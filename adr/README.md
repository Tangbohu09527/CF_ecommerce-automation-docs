# Architecture Decision Records

> 文档状态：正式 ADR 索引
> 最近复核：2026-09-03

本目录提供企业自动化项目的技术决定导航。按照仓库固定规则，[技术决策记录](../05_技术决策记录.md)仍是决定正文和状态的唯一权威来源；本目录不复制完整决定，以避免两套 ADR 分叉。

## 决定索引

| 主题 | 决定编号 | 简要边界 |
| --- | --- | --- |
| Agent 与模型 | D001-D002 | 生产 Agent 使用 Hermes；模型计划使用 GPT-5.6 API，实际接入仍需实施 |
| 微信与控制中心 | D003-D005 | `agent-wechat`、Debian 部署边界和 CFserver/PostgreSQL 权威状态 |
| 阶段与仓库 | D006-D010 | 第一阶段不建独立 OCR、FileBrowser 并行、`CF_` 命名和 GitHub 私密仓库 |
| 身份与 File Service | D011-D018 | 企业身份主键、唯一正式 File Service、capability、Token 和 Persistent Audit |
| 生产 Runtime | D019-D025 | 禁用 VNC、`cf-internal`、Persist-first、三个 Worker、首次 bootstrap、Secret 和闭环口径 |
| Profile、媒体与恢复 | D026-D031 | Hermes Profile 所有权、Attachment/Artifact、`uncertain`、`group_sender` 和分层恢复 |
| 生产 Runtime 收口 | D032-D039 | forced QR、Controller、immutable Release、Admin recovery、日志策略、Git authority、FileBrowser 和阶段口径 |

完整原因、影响和取代关系见[技术决策记录](../05_技术决策记录.md)。

## 使用规则

1. 固定技术选择发生变化时，先在 `05_技术决策记录.md` 新增决定，不直接改写旧决定的历史事实。
2. 新记录使用下一个 `Dxxx` 编号，包含日期、状态、决定、原因和影响。
3. 旧决定被取代时，由新记录明确指向旧编号；旧行保留。
4. 决定形成后，再更新范围、需求、设计、部署、运维和验证文档。
5. 提议、讨论和实验不进入决定表；先保留在 Issue 或受控验证记录中。
6. 生产证据不会自动改变决定状态；需要变更仍必须经过 ADR 流程。

## 相关正式文档

- [System Architecture](../architecture/system-architecture.md)
- [Deployment Guide](../deployment/deployment-guide.md)
- [Recovery Runbook](../operations/recovery-runbook.md)
- [Production Validation Checklist](../validation/production-validation-checklist.md)
- [2026-09-03 Production Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)
