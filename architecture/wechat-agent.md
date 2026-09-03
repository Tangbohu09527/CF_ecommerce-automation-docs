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

## Git 边界

- `main`：`92393bc2ae1d89dae9449fc131413979aa2fa2f2`。
- PR #1：`feat/forced-qr-login`，OPEN。
- PR #4：`codex/forced-qr-production-hardening-r2`，OPEN，堆叠在 PR #1。
- PR #5：未合并的组件文档工作。

生产行为已验证不等于实现已经进入 `main`。现场 Image ID 也不得在缺少构建证据时绑定到 PR #4 exact SHA。PR #4/#5 的 GitHub checks 当前部分失败。

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
