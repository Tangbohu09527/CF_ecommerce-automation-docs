# Validation 文档索引

> 文档状态：当前验证入口
>
> 最近复核：2026-09-03

## 当前入口

| 文档 | 用途 |
| --- | --- |
| [2026-09-03 Production Closeout](./records/2026-09-03-enterprise-runtime-production-closeout.md) | 当前生产 Runtime 的版本、验收、回滚和证据边界 |
| [Production Validation Checklist](./production-validation-checklist.md) | 已完成摘要与未来 Release 模板 |
| [当前状态矩阵](../status/current-status.md) | 实现、部署、生产验证和剩余边界 |
| [执行记录规范](./records/README.md) | 新验证记录命名与安全要求 |
| [历史证据索引](./history/README.md) | 2026-08-14 及更早材料 |
| [文档审计](./documentation-audit.md) | 全仓一致性、链接和 stale statement 审计 |

## 当前证据结论

- Gateway V2/P1、私聊/群聊文本、Checkpoint generation/rebase、forced QR、CFserver 核心 reboot 恢复、AI host reachability、Gateway-only cutover 和回滚证据已完成。
- V2 `group_sender` 已实现/测试/部署，但同群多发送者尚未单独生产验收。
- Context/Admin recovery 已实现、测试并随 Gateway 部署；完整动作覆盖仍需留证。
- FileBrowser V1 Beta 已实现和自动化验证，尚未生产部署。
- 完整媒体、Skills、业务系统、PostgreSQL restore、automatic boot stop gate 和 Hermes 长期 HA 未完成。

只有带日期的生产记录可以支撑 Production validated。代码、CI、healthy 或未合并 PR 都不能替代。
