# 同步架构

> **2026-10-03 更新：全团统一改用「Muse 文件交换」为主通道。**
> 本地 Agent 通过 muse 工具（`muse_put / muse_list / muse_get / muse_delete`）直接读写助理云电脑工作区
> `workspace/v5-exchange/{inbox,outbox,archive}`：inbox = Rex → Rocky，outbox = Rocky → Rex。
> 不需要同步软件、不需要在云电脑上配凭证，复用已打通的 muse 通道（含 Cookie 自动续期）。
> 单次请求上限约 45KB，工具自动按 32KB 分块；速度约 0.1MB/s，>20MB 走本仓库。规范见团队技能 `muse-exchange`。
>
> **Rex ↔ Rocky 已切换（2026-10-03）**：CEO 与 Rocky 本人确认后，Rocky 已采用 v5-exchange，Rex→Rocky 往返验收通过。
> Syncthing 已停用（VPS1 端已停止、无自启；`rocky-work/rocky-work/` 下的历史文件保留可查阅）。
> 本仓库保留为 >20MB 大文件与需要版本记录的交付物通道。

## 附：2026-10-01 方案（Syncthing 为主，GitHub 为辅）——已于 2026-10-03 停用，留作历史参考

## 1. 总览

```
                    ┌──────────── 主通道：Syncthing（实时、私有、点对点）────────────┐
                    │                                                              │
┌───────────────────┴──────────┐   TLS1.3 · VM 经出口代理 CONNECT    ┌─────────────┴──────────────┐
│ VPS1（Rex / V5DOBOT）         │ ◄──────────────────────────────────► │ Muse 云电脑 VM（Rocky）     │
│ /root/workspace/rocky-work/  │        文件夹 ID: rocky-work         │ ~/workspace/rocky-work/    │
│   rocky-work/   ← 同步根      │                                      │   rocky-work/   ← 同步根    │
│                              │                                      │                            │
│ /root/workspace/             │       辅通道：Git（异步、有版本）      │ ~/workspace/               │
│   Rex_Rocky_work/  (clone) ──┼──── push/pull ──► GitHub ◄── pull/push┼── Rex_Rocky_work/ (clone)  │
└──────────────────────────────┘     v5do/Rex_Rocky_work · main       └────────────────────────────┘
```

两条通道**各管各的文件，同一份文件只走一条通道**。

| | 主通道：Syncthing | 辅通道：GitHub 仓库（本仓库） |
|---|---|---|
| 放什么 | 日常工作文件：线索（`leads/`）、数据、分析报告、草稿 | 任务交接：`briefs/`、`outputs/`、`handoff/`；需要版本记录的规范文档 |
| 同步方式 | 实时、自动、双向 | 手动 `commit` + `push`，对方 `pull` |
| 数据路径 | 两台机器点对点，不经第三方存储 | 经过 GitHub |
| 敏感数据 | ✅ 可以放（私有通道） | ❌ 公开期严禁；转私有后也只放必要内容 |
| 版本/追溯 | 只有 `.stversions` 旧版本 | 完整 `git log`，可回滚 |

## 2. 主通道：Syncthing

| | VPS1（Rex） | VM（Rocky） |
|---|---|---|
| 同步根目录 | `/root/workspace/rocky-work/rocky-work/` | `~/workspace/rocky-work/rocky-work/` |
| 常驻方式 | syncthing 进程（Rex 维护） | systemd `syncthing-rocky.service`，开机自启 |
| 出网 | 公网直接监听 22000 | VM 沙箱拦截直连出站，Syncthing 通过 `ALL_PROXY` 走出口代理 CONNECT 到 VPS1:22000 |

**两端根目录同构**：同一个文件在两边的绝对路径后半段完全一致，例如
`…/rocky-work/rocky-work/leads/xxx.json`。2026-10-01 已用测试文件验证落点正确。

### 历史问题（已修复，留作排障参考）

| 时间 | 问题 | 处理 |
|---|---|---|
| 09-29 ~ 10-01 | 两端根目录差一层（VM 根在 `~/workspace/rocky-work/`），双方写的路径落点不一致 | VM 根目录改为 `~/workspace/rocky-work/rocky-work/`，与 VPS1 同构 |
| 09-29 17:06 起 | VM 断线约 40 小时未恢复 | 根因是 VM 出站 TCP 被沙箱拦截；自建 Python 中转桥与 Syncthing v2 的 reuse-port 拨号冲突导致回包错乱。改用 Syncthing 原生代理支持，并用 systemd 常驻 |
| 10-01 | VPS1 同步根目录上一层残留旧的 `.stfolder` 与测试文件 | 已清理。注意 VPS1 Syncthing 的**新建文件夹默认路径**仍是 `/root/workspace/rocky-work`，若重新添加文件夹，需手动改成 `…/rocky-work/rocky-work` |

### 排障速查

- 在 VPS1 上看连接：`curl -s -H "X-API-Key: <key>" http://127.0.0.1:8384/rest/system/connections`，Rocky VM 的 `connected` 应为 `true`。
- 文件没过来：先看两端是否连接，再确认文件在**同步根目录之内**（根目录之外的文件不会同步）。
- 出现 `*.sync-conflict-*` 文件：说明两边同时改了同一个文件，人工合并后删除冲突副本。

## 3. 辅通道：GitHub 仓库

- 唯一事实源：`github.com/v5do/Rex_Rocky_work` 的 `main` 分支。
- 克隆位置：VPS1 `/root/workspace/Rex_Rocky_work`，VM `~/workspace/Rex_Rocky_work`。
- **仓库克隆目录绝不能放在 Syncthing 同步根目录里**，否则 `.git` 会被双向同步而损坏。两者在 `workspace/` 下并列。
- 可选：定时只拉不推。
  ```bash
  */5 * * * * cd ~/workspace/Rex_Rocky_work && git pull --rebase --autostash -q >> ~/.rex_rocky_pull.log 2>&1
  ```

## 4. 访问与认证

| | Syncthing | GitHub |
|---|---|---|
| Rex（VPS1） | 设备 ID 配对 | 只授权本仓库的 fine-grained token（`Contents: Read and write`） |
| Rocky（VM） | 设备 ID 配对；代理凭据只在 service 文件里，权限 600 | 单独一个 fine-grained token，不与 Rex 共用 |

凭证只放在各自机器的配置 / git credential 中，**不写进任何同步文件、仓库、日报或聊天**。

## 5. 改为私有库

1. GitHub → Settings → General → Change visibility → Private。
2. 双方各 `git pull` / `git push` 一次，确认凭证可用。
3. 公开期间推送过的内容视为已公开；误传过敏感信息需要轮换相关凭证，删除文件不够。
