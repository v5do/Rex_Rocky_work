# 协同规范与注意点（Rex × Rocky）

## 一、公开期红线

仓库当前**公开**，推送即公开，历史记录永久保留（删除文件也能从历史里找回）。改为私有前：

1. ❌ 不放真实客户的姓名、公司、邮箱、电话、WhatsApp、地址。线索文件用样例数据，例如 `Customer A / example.com / +00 000`。
2. ❌ 不放报价、成本、毛利、订单金额、合同、发票。
3. ❌ 不放任何 API key、token、密码、cookie、`.env`、私钥、配置文件。
4. ❌ 不放内部 IP、服务器地址、内部系统账号。
5. ✅ 每次推送前执行一次自查（见第五节命令），确认无上述内容。

误推了敏感内容：**立即告诉 Rex 和 CEO**，不要自己用 force push 改历史。

## 二、分工与写入范围（避免互相覆盖）

| 目录 | 谁写 | 谁读 | 说明 |
|---|---|---|---|
| `briefs/` | Rex | Rocky | 任务说明，一个任务一个文件：`YYYYMMDD-任务简称.md` |
| `outputs/` | Rocky | Rex | 交付物，文件名与对应 brief 一致，加后缀：`YYYYMMDD-任务简称-v1.md` |
| `leads/` | 双方 | 双方 | 一条线索/一批线索一个文件，**不要两人同时改同一个文件** |
| `handoff/HANDOFF.md` | 双方 | 双方 | 只在末尾**追加**，不修改、不删除对方写的行 |
| `docs/` | 双方协商 | 双方 | 改动前先在 HANDOFF 里说明 |

原则：**自己的目录随便改，对方的目录只读**。需要对方修改时，在 HANDOFF 里留言，而不是直接改对方的文件。

## 三、标准工作流

```bash
cd ~/workspace/Rex_Rocky_work      # VPS1 上是 /root/workspace/Rex_Rocky_work
git pull --rebase                   # 1. 开工前先拉最新
# 2. 在自己负责的目录里新建/修改文件
git status                          # 3. 确认只改了该改的文件
git add briefs/20261001-xxx.md      # 4. 逐个 add，不要用 `git add -A` 一把梭
git commit -m "[Rex] 新增任务：xxx"  # 5. 提交信息以 [Rex] 或 [Rocky] 开头
git pull --rebase && git push       # 6. 推送前再拉一次，再推
```

然后在 `handoff/HANDOFF.md` 末尾追加一行，告诉对方有新东西（格式见该文件）。

## 四、冲突与异常处理

| 情况 | 处理 |
|---|---|
| `git push` 被拒（rejected / non-fast-forward） | 对方先推了。执行 `git pull --rebase` 后再 `git push`。 |
| `pull --rebase` 提示冲突 | 冲突文件里会有 `<<<<<<<` 标记。如果是对方目录的文件：`git checkout --theirs <文件>`；如果双方都改了同一文件：保留双方内容合并后 `git add` → `git rebase --continue`。拿不准就 `git rebase --abort` 并在 HANDOFF 里求助。 |
| 想撤回刚推的内容 | 用 `git revert <提交>` 生成反向提交，**禁止 `git push --force`**。 |
| 认证失败（401/403） | 凭证过期或被吊销，联系 Rex/CEO，不要换用别人的凭证。 |
| 大文件 | 单文件不超过 20MB，不提交压缩包、安装包、视频。需要时放云盘，仓库里只留链接说明。 |

## 五、推送前自查命令

```bash
# 列出待推送的改动里疑似敏感的内容；有输出就逐条确认
git diff --cached | grep -nEi 'api[_-]?key|token|secret|passw|密码|sk-[a-z0-9]{16}|BEGIN .*PRIVATE KEY|@[a-z0-9-]+\.(com|cn|net|de)|\+?[0-9]{10,}'
```

## 六、提交信息约定

```
[Rex]   新增任务：东南亚医疗器械经销商线索整理
[Rocky] 交付：20261001-东南亚经销商-v1（12 条样例线索）
[Rocky] 修订：按 Rex 意见补充联系渠道字段
```

## 七、试运行验证步骤

1. Rex：在 `handoff/HANDOFF.md` 追加 “Rex → Rocky 测试”，提交推送。
2. Rocky：`git pull --rebase`，确认能看到这一行，在其下追加回复，提交推送。
3. Rex：`git pull --rebase`，确认看到 Rocky 的回复。
4. 双方各在自己的目录建一个测试文件，再互相拉取确认。
5. 都成功 → 在 HANDOFF 记录“验证通过”，报 CEO 申请改为私有库。
