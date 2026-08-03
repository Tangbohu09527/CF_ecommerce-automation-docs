# 电商业务全自动化系统文档

> 状态标识：**已确定**表示当前有效基线；**待技术验证 / 待确认**表示尚不能作为稳定能力承诺；**后续规划**表示不属于当前阶段。

## 仓库用途

本仓库集中维护“电商业务全自动化系统”的项目总纲、功能需求、系统设计、开发规范、部署运维和技术决策，不存放业务代码或真实业务文件。

**一句话目标：** 让员工通过微信提交文字与附件任务，由受控的 AI 和自动化工具完成处理，并把可追踪的结果返回原会话。

## 当前固定技术方案

- **已确定：** Debian 是消息、上下文、任务、文件、权限、日志和审计的权威控制中心。
- **已确定：** 生产 Agent 使用 Hermes，模型计划使用 GPT-5.6 API，不使用 OpenClaw。
- **已确定：** 微信接入采用 `agent-wechat`，计划运行在 Debian；它只负责微信收发和附件获取。
- **已确定：** 新 Windows AI 电脑运行 Hermes、Worker Bridge、Skills 及文档、浏览器和 Windows 自动化工具。
- **已确定：** `CF_filebrowser-enterprise` 是正式企业 File Service；FileBridge / `filebrowser-agentctl` 是受控客户端，Gateway、Hermes、Skills 和客户端均不得绕过其 API、文件权限、Token capability、Share capability 或 Audit。
- **已确定：** 第一阶段不建设独立 OCR；自动化主线与 FileBrowser Enterprise 二次开发并行推进。
- **已实现：** File Service 已实现以下 V1 Beta 服务端能力：Token hash-only 与一次性明文、旧 Token/Key V5 强制迁移、文件 capability 与 Archive/Extract 安全、Share 服务端 capability 契约，以及持久化 Audit、Token Audit 和管理员查询 API。
- **待集成 / 待验证：** Share capability 前端 UI、其余业务 Audit Action、Gateway/Hermes 稳定契约对接、最终 Debian 部署验收和 V1 Beta 最终 tag/candidate 尚未完成；当前不代表正式生产上线。
- **已验证：** `agent-wechat` V1 入口已完成微信登录、私聊文本、群聊文本、文件消息、ZIP 文件、引用消息、`sender` 识别、`chatId` 识别和文件获取验证；合并转发消息已验证类型识别、发送人获取和外层标题获取。
- **待技术验证 / 待开发：** 图片、Office、PDF、中文文件名、连续多附件、失败重试和长期运行稳定性等未覆盖场景，以及合并转发内部聊天记录展开和内部文件自动提取。

## 文档入口

| 文档 | 内容 |
| --- | --- |
| [项目总纲](./00_项目总纲.md) | 范围、总体模块、设备分工、建设阶段 |
| [功能需求](./01_功能需求.md) | 系统要做什么及验收边界 |
| [系统设计](./02_系统设计.md) | 架构、职责、数据流、状态与安全边界 |
| [员工工作区与 AI 会话线程设计](./design/employee-workspace-design.md) | 企业身份、员工工作区、AI 线程隔离与 Hermes 运行时绑定 |
| [开发规范](./03_开发规范.md) | 仓库、Git、数据与文档规则 |
| [部署运维](./04_部署运维.md) | 测试环境基线、待部署服务与排障原则 |
| [技术决策记录](./05_技术决策记录.md) | 当前有效决定及其原因和影响 |
| [AI 系统总体架构](./architecture/ai-system-overview.md) | 面向企业业务的分层架构和当前实现边界 |
| [agent-wechat 定位](./architecture/wechat-agent.md) | 微信入口层职责、非职责和接口边界 |
| [消息与任务流程](./architecture/message-flow.md) | 普通问答、企业业务和规划中文件处理流程 |
| [组件职责图谱](./architecture/component-map.md) | 核心组件职责、状态和系统关系 |
| [当前开发进度](./status/current-progress.md) | 已验证事实、待完成工作和规划能力 |
| [agent-wechat V1 入口验证记录](./status/agent-wechat-validation.md) | 验证时间、范围、已完成和未完成能力 |
| [AI 协作入口](./AGENTS.md) | 后续 Codex 和其他代码 AI 的约束 |

## 相关仓库

| 仓库 | 状态 |
| --- | --- |
| `CF_ecommerce-automation-docs` | **已确定：** 当前文档仓库 |
| `CF_agent-gateway` | **已实现基础：** Python 3.12 + FastAPI 工程和 Message Store Foundation 已完成；其余 Gateway 模块与部署验证仍待完成 |
| `CF_filebrowser-enterprise` | **已实现：** 正式企业 File Service 的 V1 Beta 核心服务端能力已进入 `feat/v1-integration`；自动化对接与最终 Debian 部署仍为**待集成 / 待验证**，本仓库任务不得修改其实现 |
| 其他 `CF_` 前缀代码仓库 | **后续规划：** 名称与边界须另行确认 |

## 当前状态

当前仍处于**阶段 1**。Windows AI 节点与 Hyper-V Debian 测试节点的基础运行环境已经完成，`agent-wechat` V1 微信入口验证已经完成；合并转发消息的类型、发送人和外层标题识别也已补充验证，但内部聊天记录展开与内部文件自动提取尚未支持。Gateway 架构、Message Store、Access Control、Task Queue、Hermes 事件协议以及员工工作区与 AI Thread 已形成设计基线。`CF_agent-gateway` 已完成 Python 3.12 + FastAPI 工程基础和 Message Store Foundation，包括 Conversation / Message / Attachment、消息写入与查询及 `event_id` 幂等；Identity Mapping、Employee Conversation Manager、Access Control、Context Builder、Task Queue、Adapter 与 Hermes 链路仍待实现。`CF_filebrowser-enterprise` 已实现 V1 Beta 核心服务端能力，但尚未完成全部业务 Audit Action、自动化主线集成、最终 Debian 部署验收或正式生产上线。Gateway 已提供 Dockerfile 和 Compose 配置，但尚未完成 Docker 镜像构建和部署验证。设计完成或代码基础完成均不代表生产上线。详见[当前开发进度](./status/current-progress.md)、[agent-wechat V1 入口验证记录](./status/agent-wechat-validation.md)和[部署运维](./04_部署运维.md)。
