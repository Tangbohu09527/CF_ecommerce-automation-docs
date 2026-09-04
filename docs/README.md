# docs 兼容与历史索引

> 状态日期：2026-09-03
>
> 根级 `architecture/`、`deployment/`、`operations/`、`validation/`、`status/` 和 `05_技术决策记录.md` 是正式权威体系。`docs/` 不再维护第二份当前生产事实。

## 当前权威入口

| 主题 | 文档 |
| --- | --- |
| 当前状态 | [status/current-status.md](../status/current-status.md) |
| 当前进度 | [status/current-progress.md](../status/current-progress.md) |
| 系统架构 | [architecture/system-architecture.md](../architecture/system-architecture.md) |
| 部署 | [deployment/deployment-guide.md](../deployment/deployment-guide.md) |
| 恢复 | [operations/recovery-runbook.md](../operations/recovery-runbook.md) |
| 验证 | [validation/production-validation-checklist.md](../validation/production-validation-checklist.md) |
| 生产证据 | [2026-09-03 Production Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md) |

## 兼容入口

| 路径 | 用途 |
| --- | --- |
| [overall-architecture.md](./architecture/overall-architecture.md) | 旧总体架构链接 |
| [project-boundaries.md](./architecture/project-boundaries.md) | 旧职责边界链接 |
| [production-topology.md](./deployment/production-topology.md) | 旧生产拓扑链接 |
| [v1-current-status.md](./status/v1-current-status.md) | 旧 V1 状态链接 |

这些页面只做简短导航，不复制完整状态。

## Historical / Archived

以下材料保留当时日期、SHA、环境和限制，不是当前 Runbook：

- [2026-08-11 V2 Enterprise Runtime](./architecture/v2-enterprise-runtime.md)
- [2026-08-11 CFserver Staging](./deployment/cfserver-staging-status.md)
- [2026-08-11 Staging Debian](./deployment/staging-debian.md)
- [2026-08-11 Runtime Operations](./operations/runtime-operations.md)
- [2026-08-11 Current Limitations](./status/current-limitations.md)
- [2026-08-11 Context Runtime](./context/context-runtime.md)
- [2026-08-11 Admin API](./admin/admin-api.md)

历史材料中的“当前”“未启用”“版本”和命令只按文件日期理解。Gateway 当前 Context/Admin、P1、forced QR、重启和 FileBrowser 状态一律以根级权威入口为准。
