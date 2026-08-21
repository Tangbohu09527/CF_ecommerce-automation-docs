# docs 文档索引

> 文档更新日期：2026-08-21
>
> 当前状态证据基线：2026-08-14
>
> 正式生产文档已经收口到仓库根级的 `architecture/`、`deployment/`、`operations/`、`validation/`、`adr/` 五个目录。本 `docs/` 目录仅保留兼容系统级摘要和 2026-08-11 Gateway V2 Enterprise Runtime 历史快照；两类材料的状态日期与适用范围不得混用。

## 正式生产文档

| 文档 | 内容 |
| --- | --- |
| [System Architecture](../architecture/system-architecture.md) | 企业自动化总体架构、职责、信任边界和端到端主链 |
| [Deployment Guide](../deployment/deployment-guide.md) | 新机器、Docker、配置、升级和回滚 |
| [Timezone Policy](../deployment/timezone-policy.md) | Host、Container、Database 与 Application 时区基线 |
| [Recovery Runbook](../operations/recovery-runbook.md) | Docker、微信、Hermes 和消息无回复的故障恢复 |
| [Production Validation Checklist](../validation/production-validation-checklist.md) | 部署、升级、回滚和恢复后的验收与证据模板 |
| [ADR 索引](../adr/README.md) | 技术决定主题导航；决定正文仍以根目录技术决策记录为唯一权威 |
| [当前状态矩阵](../status/current-status.md) | 当前能力状态、证据、限制和下一动作 |

## 兼容系统级摘要

| 文档 | 内容 |
| --- | --- |
| [整体架构兼容摘要](./architecture/overall-architecture.md) | 既有系统级架构链接的兼容摘要 |
| [项目边界兼容摘要](./architecture/project-boundaries.md) | 既有组件职责边界链接的兼容摘要 |
| [V1 状态兼容摘要](./status/v1-current-status.md) | 证据截止 2026-08-14 的状态摘要，不产生新运行证据 |
| [生产拓扑兼容摘要](./deployment/production-topology.md) | 当前部署事实和未来规划摘要，不是部署操作手册 |

上述文件仅用于兼容既有链接和建立系统级认知，不取代根级正式文档。当前生产状态和实机证据以[当前状态矩阵](../status/current-status.md)及其验证记录为准，固定技术决定以[技术决策记录](../05_技术决策记录.md)为准；本目录不复制组件实现细节或运行命令。

## V2 Enterprise Runtime 历史快照

> 适用版本：`v2-enterprise-runtime-20260811`
> 文档状态：2026-08-11 Staging 版本化快照

除上表四份兼容系统级摘要外，原有 `docs/` 内容保留 `CF_agent-gateway` V2 Enterprise Runtime 在 2026-08-11 的架构、Staging 部署、运维、Context Runtime、Admin Archive API 和已知限制，供版本追溯与历史交接使用。

历史文件中的“当前”“已启用”“未启用”均按 2026-08-11 的 Staging 环境理解，不能覆盖根目录当前状态，也不能覆盖本轮新增系统级摘要中明确标注的较新工作项。

## 正式文档阅读顺序

1. [System Architecture](../architecture/system-architecture.md)
2. [当前状态矩阵](../status/current-status.md)
3. [Deployment Guide](../deployment/deployment-guide.md)
4. [Timezone Policy](../deployment/timezone-policy.md)
5. [Recovery Runbook](../operations/recovery-runbook.md)
6. [Production Validation Checklist](../validation/production-validation-checklist.md)
7. [ADR 索引](../adr/README.md)与[技术决策记录](../05_技术决策记录.md)

## 历史快照阅读顺序

1. [V2 Enterprise Runtime 架构总览](./architecture/v2-enterprise-runtime.md)
2. [Staging Debian 部署](./deployment/staging-debian.md)
3. [CFserver Staging 部署状态](./deployment/cfserver-staging-status.md)
4. [Runtime 运维](./operations/runtime-operations.md)
5. [Context Runtime](./context/context-runtime.md)
6. [Admin API](./admin/admin-api.md)
7. [2026-08-11 当前限制快照](./status/current-limitations.md)

## 内容归属

| 主题 | 权威文档 |
| --- | --- |
| 企业自动化总体架构与信任边界 | [System Architecture](../architecture/system-architecture.md) |
| 当前生产状态、限制与下一步 | [当前状态矩阵](../status/current-status.md) |
| 当前系统级摘要 | [V1 当前状态](./status/v1-current-status.md) |
| 当前总体范围与设备职责 | [项目总纲](../00_项目总纲.md) |
| 当前组件职责与数据流 | [系统设计](../02_系统设计.md) |
| 当前系统级生产运维 | [根目录部署运维](../04_部署运维.md) |
| 固定技术决定 | [技术决策记录](../05_技术决策记录.md) |
| 2026-08-14 文本、引用和图片发现证据 | [私聊、群聊及媒体验证记录](../status/2026-08-14-private-group-media-validation.md) |
| 2026-08-11 V2 模块设计快照 | [历史架构总览](./architecture/v2-enterprise-runtime.md) |
| 2026-08-11 CFserver Staging 事实 | [历史 Staging 部署状态](./deployment/cfserver-staging-status.md) |

## 历史快照边界

- 历史文件不会被改写成当前底层实现文档。
- 其中“Workers 未启用”“Hermes 尚未接入”等表述是 2026-08-11 当日 Staging 快照，不是当前生产结论。
- 当前统一结论仍是：私聊和 `group_sender` 群聊的授权文本闭环已实机验证；媒体链路、引用上下文注入和完整宿主恢复仍待完成。
