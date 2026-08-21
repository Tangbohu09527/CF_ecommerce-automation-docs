# Architecture 文档索引

> 文档状态：当前生产文档入口
> 最近复核：2026-08-21
> 运行证据截止：2026-08-14

本目录维护“电商业务全自动化系统”的现行企业架构。架构文档描述职责、边界和已确认的数据流；能力是否已经实机验证，始终以[当前状态矩阵](../status/current-status.md)和对应验证记录为准。

## 正式架构

| 文档 | 用途 |
| --- | --- |
| [System Architecture](./system-architecture.md) | 企业自动化总体架构、部署边界、信任边界和端到端主链 |
| [WeChat Runtime Design](./wechat-runtime-design.md) | Polling、Checkpoint、`localId`、恢复和 at-least-once 语义 |
| [Gateway 架构](./gateway-architecture.md) | Gateway 内部组件、权威状态和执行边界 |
| [消息与任务流程](./message-flow.md) | 当前文本流程以及引用、媒体和 Artifact 目标流程 |
| [微信与 Hermes 集成](./wechat-hermes-integration.md) | `agent-wechat`、Gateway Worker 与 Hermes 的集成边界 |
| [agent-wechat 职责](./wechat-agent.md) | 微信入口负责与不负责范围 |
| [组件图谱](./component-map.md) | 组件级状态摘要 |

## 权威关系

- [System Architecture](./system-architecture.md)是企业级架构入口。
- [系统设计](../02_系统设计.md)完整记录组件职责、数据流、状态和数据对象。
- [技术决策记录](../05_技术决策记录.md)优先于较早蓝图和设计快照。
- [当前状态矩阵](../status/current-status.md)记录能力状态；架构图中的目标节点不构成完成证据。
- `design/` 和 `docs/` 中标明日期的材料按其日期阅读，不作为当前生产操作手册。
