# 同步架构

## 1. 总览

```
┌──────────────────────────┐        git push / pull        ┌──────────────────────────┐
│ VPS1（Rex / V5DOBOT）     │ ───────────────►  GitHub  ◄─── │ Muse 云电脑 VM（Rocky）   │
│ /root/workspace/          │      HTTPS / SSH       main    │ ~/workspace/             │
│   Rex_Rocky_work/  (clone)│ ◄───────────────               │   Rex_Rocky_work/ (clone)│
└──────────────────────────┘                                 └──────────────────────────┘
              ▲                                                            ▲
              └────── 唯一事实源：github.com/v5do/Rex_Rocky_work 的 main 分支 ──┘
```

- **没有直连**：VPS1 与 VM 之间不需要能互相访问，只要各自能访问 GitHub。
  VM 走 Cloudflare WARP 出网，IP 会变；Git 方案不受影响。
- **异步交换**：一方提交并推送后，另一方下一次 `pull` 时拿到。不是实时同步。
- **每次变更有记录**：谁、什么时候、改了什么，都能在 `git log` 里查到，可回滚。

## 2. 为什么从 Syncthing 改为 Git

此前两边用 Syncthing 同步 `rocky-work` 文件夹，2026-09-30 核查发现：

| 问题 | 详情 |
|---|---|
| 两端根目录差一层 | VPS1 根 = `/root/workspace/rocky-work/rocky-work/`，VM 根 = `~/workspace/rocky-work/`。双方都写 `…/rocky-work/rocky-work/leads/`，实际落到对方的位置不一样，看起来就是“没同步”。 |
| 连接会断且不自愈 | VM 端 2026-09-29 17:06 断开后一直未重连（VM 休眠/重启后 Syncthing 未自启），期间两边各写各的。 |
| 无版本记录 | 冲突时生成 `.sync-conflict` 文件，无法知道谁改了什么。 |

Git 方案中，路径永远是**相对仓库根目录**的（`leads/xxx.md` 在两边都是同一个文件），不存在层级错位。

## 3. 与 Syncthing 的关系（重要）

- **仓库克隆目录不能放在 Syncthing 同步目录里面。** `.git` 被 Syncthing 双向同步会损坏仓库。
  所以克隆到 `~/workspace/Rex_Rocky_work`，与旧的 `~/workspace/rocky-work` 并列，而不是放进去。
- 试运行期间 Syncthing 可以继续保留，但**同一份文件只走一条通道**：凡是放进本仓库的文件，不再放进 `rocky-work`。
- 方案确认后，停用 `rocky-work` 的 Syncthing 共享，把仍需要的文件迁入本仓库。

## 4. 访问与认证

| | 认证方式 | 要求 |
|---|---|---|
| Rex（VPS1） | v5do 账号的 fine-grained PAT 或 SSH 密钥 | 只授权本仓库 `Contents: Read and write` |
| Rocky（VM） | 单独一个 fine-grained PAT（仅本仓库）或 Deploy key（勾选 write） | 不与 Rex 共用凭证，方便单独吊销 |

- 凭证只存放在各自机器的 git credential helper / `~/.ssh` 中，**绝不写进仓库文件、提交信息、日报或聊天记录**。
- 改为私有库后公开访问失效，以上凭证继续有效（前提是 PAT 授权了该仓库）。

## 5. 自动拉取（可选）

为让对方的更新尽快可见，可各自加一个定时拉取，只拉不推：

```bash
# 每 5 分钟拉取一次；有本地未提交修改时 rebase 会安全失败，不会覆盖本地文件
*/5 * * * * cd ~/workspace/Rex_Rocky_work && git pull --rebase --autostash -q >> ~/.rex_rocky_pull.log 2>&1
```

**不要自动 `commit`/`push`**：只在一项工作完成、自查无敏感信息后，手动提交推送。

## 6. 改为私有库的步骤

1. GitHub → Settings → General → Danger Zone → Change visibility → Private。
2. 双方各执行一次 `git pull` / `git push`，确认凭证仍可用。
3. 公开期间推送过的内容视为已公开：若误传过敏感信息，**删文件不够**，需要轮换相关密钥/密码，并评估客户信息外泄影响。
