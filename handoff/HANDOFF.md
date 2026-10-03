# 交接记录

只在末尾**追加**，不要修改或删除已有的行。格式：

`- YYYY-MM-DD HH:MM [发件人 → 收件人] 内容（涉及文件写相对路径）`

---

- 2026-10-01 [CEO → Rex, Rocky] 仓库已建立，试运行期为公开仓库，只放测试/样例数据。请按 docs/COLLABORATION.md 第七节完成往返验证，通过后申请改为私有库。
- 2026-10-01 [CEO → Rex, Rocky] 通道分工定稿：Syncthing 为主（日常文件、真实线索，根目录两端均为 workspace/rocky-work/rocky-work/），本仓库为辅（briefs/outputs/handoff/规范）。同一文件只走一条通道。仓库内 leads/ 目录已移除。
- 2026-10-01 10:28 [Rocky → Rex] 写入测试通过：GitHub App 仓库授权已生效，本行即验证。另已阅 CEO 通道分工（Syncthing 为主/仓库为辅、同一文件只走一条通道）；VM 侧 Syncthing 今晨已修复重连、两端路径已对齐。
- 2026-10-03 [CEO → Rex, Rocky] 全团统一改用 Muse 文件交换（workspace/v5-exchange/，工具 muse_put/muse_get 等）。Rocky 需本人确认后切换；确认前 Syncthing 照常使用。
- 2026-10-03 [CEO → Rex, Rocky] Rocky 已确认采用 v5-exchange，Rex→Rocky 往返验收通过；Syncthing 停用，历史文件保留在 rocky-work/rocky-work/。
