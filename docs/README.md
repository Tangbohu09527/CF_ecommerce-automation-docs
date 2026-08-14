# CF_agent-gateway V2 Enterprise Runtime 历史文档索引

> 适用版本：`v2-enterprise-runtime-20260811`
> 文档状态：2026-08-11 Staging 版本化快照
> 当前生产状态：请先阅读[当前状态矩阵](../status/current-status.md)与[2026-08-14 私聊、群聊及媒体验证记录](../status/2026-08-14-private-group-media-validation.md)

## 文档范围

本目录保留 `CF_agent-gateway` V2 Enterprise Runtime 在 2026-08-11 的架构、Staging 部署、运维、Context Runtime、Admin Archive API 和已知限制，供版本追溯与历史交接使用。

目录中的“当前”“已启用”“未启用”均按 2026-08-11 的 Staging 环境理解，不能覆盖根目录当前状态。当前私聊和 `group_sender` 群聊授权文本闭环已实机验证；媒体桥、引用上下文注入和完整宿主恢复的边界以最新状态记录为准。

固定技术决定以根目录的[技术决策记录](../05_技术决策记录.md)为准。具体 Gateway 生产部署和运行命令以 `CF_agent-gateway` 项目仓库的当前文档为准，本目录不作为最新命令来源。

## 阅读顺序

1. [当前状态矩阵](../status/current-status.md)
2. [2026-08-14 私聊、群聊及媒体验证](../status/2026-08-14-private-group-media-validation.md)
3. [2026-08-13 微信运行时收口](../status/2026-08-13-wechat-runtime-closeout.md)
4. [V2 Enterprise Runtime 架构总览](./architecture/v2-enterprise-runtime.md)
5. [Staging Debian 部署](./deployment/staging-debian.md)
6. [CFserver Staging 部署状态](./deployment/cfserver-staging-status.md)
7. [Runtime 运维](./operations/runtime-operations.md)
8. [Context Runtime](./context/context-runtime.md)
9. [Admin API](./admin/admin-api.md)
10. [2026-08-11 当前限制快照](./status/current-limitations.md)

## 内容归属

| 主题 | 权威文档 |
| --- | --- |
| 当前生产状态、限制与下一步 | [当前状态矩阵](../status/current-status.md) |
| 2026-08-14 文本、引用和图片发现证据 | [私聊、群聊及媒体验证记录](../status/2026-08-14-private-group-media-validation.md) |
| 2026-08-13 入口与未授权拒绝证据 | [微信运行时收口记录](../status/2026-08-13-wechat-runtime-closeout.md) |
| 当前跨项目生产运维入口 | [根目录部署运维](../04_部署运维.md) |
| 2026-08-11 V2 模块设计快照 | [架构总览](./architecture/v2-enterprise-runtime.md) |
| 2026-08-11 CFserver Staging 事实 | [Staging 部署状态](./deployment/cfserver-staging-status.md) |
| 固定技术决定 | [技术决策记录](../05_技术决策记录.md) |

## 历史快照边界

- 本目录不会被改写成当前底层实现文档；它保留 2026-08-11 的版本证据。
- 其中“workers 未启用”“Hermes 尚未接入”等表述是当日 Staging 快照，不是当前生产结论。
- 2026-08-13 收口记录保留当日入口与拒绝路径事实；后续结果以 2026-08-14 记录为准。
- 当前统一结论为：“私聊和 `group_sender` 群聊的授权文本闭环已实机验证；媒体链路、引用上下文注入和完整宿主恢复仍待完成。”
