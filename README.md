# Rex_Rocky_work

Rex（V5DOBOT，VPS1）与其 Muse 助理 Rocky（Muse 云电脑 VM）的**共享工作仓库**。
两边通过 Git（GitHub）交换文件：一方 `push`，另一方 `pull`，以 GitHub 上的 `main` 分支为唯一事实源。

> ⚠️ **当前为公开仓库（试运行阶段）**：任何人都能看到这里的全部内容和历史记录。
> 试运行期间**只放测试/样例数据**，严禁提交真实客户信息、报价、密钥。详见 [协同规范](docs/COLLABORATION.md#一公开期红线)。
> 方案验证通过后改为私有库。

## 快速上手

| | Rex（VPS1） | Rocky（Muse VM） |
|---|---|---|
| 本地克隆路径 | `/root/workspace/Rex_Rocky_work` | `~/workspace/Rex_Rocky_work` |
| 开始工作前 | `git pull --rebase` | `git pull --rebase` |
| 完成一项后 | `git add <文件> && git commit -m "[Rex] …" && git push` | `git add <文件> && git commit -m "[Rocky] …" && git push` |

- 同步架构：[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- 协同规范与注意点：[docs/COLLABORATION.md](docs/COLLABORATION.md)
- 任务交接记录：[handoff/HANDOFF.md](handoff/HANDOFF.md)

## 目录结构

```
Rex_Rocky_work/
├── README.md            本文件
├── docs/                架构与规范（改动需双方确认）
├── leads/               线索文件（试运行期只放样例）
├── briefs/              Rex → Rocky 的任务说明（Rex 写，Rocky 只读）
├── outputs/             Rocky → Rex 的交付物（Rocky 写，Rex 只读）
└── handoff/             交接记录与验证测试（双方追加，不改对方的行）
```
