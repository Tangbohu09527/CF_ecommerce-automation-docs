# agent-wechat 定位与职责

> 状态日期：2026-09-03

## 当前定位

`CF_agent-wechat` 是外部微信通道 Runtime，不是 Gateway 内部模块。它负责微信进程、fresh QR、auth/chats/messages、消息读取和发送；Gateway/PostgreSQL 负责身份权限、线程、Dispatch、Response 和 Delivery 权威状态。

## 当前生产契约

| 项目 | 当前值 |
| --- | --- |
| Compose project / container | `cf-agent-wechat` |
| restart | `"no"` |
| API | 6174，loopback only |
| network | external `cf-internal`，alias `cf-agent-wechat` |
| VNC | `ENABLE_VNC=0`，无 VNC/noVNC/x11vnc/websockify |
| logs | `json-file 20m x 3` |
| Token | 独立只读 mount |
| Session | Host/container/Runtime restart 后 fresh QR；Archive 不复用 |

## 已验证行为

- forced fresh QR、手机扫码与 auth/chats/messages 放行。
- 私聊和群聊文本读取/发送。
- 图片来源发现与 JPEG 字节读取。
- CFserver reboot 后保持停止，fresh QR 后重新上线。
- Gateway-only deployment 不重建容器并保持 Session。

## Git 与镜像边界

- Repository branch authority：`main`；live tip 动态查询。
- PR #1/#4/#5/#6：全部 MERGED。
- 2026-09-04 promotion merge baseline：`02583fe76220916019ca961bb37dfa015640384e`。
- 2026-09-04 documentation post-promotion snapshot：`69f07702b6ee16d8e9700b3a53d5ebbb8ee875f8`。
- main CI Run `33863104399` completed/success，全部 Job 成功。
- observed production image ID：`sha256:7ee0309980b7d03b747b40c6c04cbaeafe2d8fc01fc9429810cbc7571ebbf720`。

Repository promotion 和 component documentation closeout 已完成，但这些合并没有重新构建或部署生产镜像。现场 Image ID 与选定 Release Commit、构建输入之间的 exact mapping 仍未证明；forced-QR 生产行为仍以 2026-09-03 验收为准。

## 不负责

- Enterprise Identity、Access Policy、Admission 和 Thread Policy。
- Hermes 调用、Context、Skills 和业务系统。
- Message/Checkpoint/Dispatch/Response/Delivery 权威状态。
- 正式企业文件权限和 Persistent Audit。

## 当前限制

- automatic boot stop gate 不由本组件保证，且尚未验证。
- agent-wechat 自身 restart 后不能复用旧 Session。
- 图片字节可读不等于 AI 支持看图或媒体回传完成。
- Archive 和备份长期保留由外部运维策略负责。

底层实现以组件仓库和 PR 栈为准；系统级操作见[Recovery Runbook](../operations/recovery-runbook.md)。
