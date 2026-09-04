# 核心项目与组件职责边界

> Compatibility entry，状态日期：2026-09-03
>
> 当前完整职责以[System Architecture](../../architecture/system-architecture.md)和[系统设计](../../02_系统设计.md)为准。

| 组件 | 当前职责 | 当前边界 |
| --- | --- | --- |
| `CF_agent-wechat` | forced QR、微信读取/发送 | 外部通道 Runtime，不负责权限、线程和 Hermes |
| `CF_agent-gateway` | Message、Admission、Thread、Context、Dispatch、Response、Delivery、Admin recovery | 与 PostgreSQL 共同构成消息和控制状态权威 |
| Hermes | Agent 与模型执行 | Windows AI 主机 external runtime，不覆盖 Gateway 状态 |
| `CF_filebrowser-enterprise` | 唯一正式 File Service | V1 Beta 已实现/自动化验证，尚未部署与集成 |
| Skills / 业务系统 | 后续确定性执行 | 尚未接入 |

当前文本链、重启和生产证据见[2026-09-03 Production Closeout](../../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。
