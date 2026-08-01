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
- **已确定：** 正式文件操作经 Debian File Service 完成权限检查和审计，Hermes 不直接拥有正式存储的任意读写权。
- **已确定：** 第一阶段不建设独立 OCR；自动化主线与 FileBrowser Enterprise 二次开发并行推进。
- **已验证：** `agent-wechat` V1 入口已完成微信登录、私聊文本、群聊文本、文件消息、ZIP 文件、引用消息、`sender` 识别、`chatId` 识别和文件获取验证；合并转发消息已验证类型识别、发送人获取和外层标题获取。
- **已验证：** 微信群结构化 mention 实测中，只有从成员列表选择当前机器人时原始 `isMentioned=true`；`@` 其他成员或只复制 / 输入机器人名称时该字段缺失。Gateway 仅按 `raw.get("isMentioned") is True` 生成 `is_mentioned`，字段缺失按 `false`。
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
| `CF_agent-gateway` | **部分实现：** main commit `f0f0ea0cbcc1029104002b566912afabd23423c7` 已实现 Message Store 来源账号隔离与双重幂等、身份 / 工作区 / 线程基础、Access Control 规则与策略持久化，以及微信适配基础组件；全量 162 项测试通过。正式接线、任务与 Hermes 链路仍待完成 |
| `filebrowser-enterprise` | **已确定：** 当前继续并行二次开发，暂不重命名；本仓库任务不得修改它 |
| 其他 `CF_` 前缀代码仓库 | **后续规划：** 名称与边界须另行确认 |

## 当前状态

当前仍处于**阶段 1**。Windows AI 节点与 Hyper-V Debian 测试节点的基础运行环境已经完成，`agent-wechat` V1 微信入口及结构化 mention 已完成限定范围验证；合并转发内部聊天记录展开与内部文件自动提取尚未支持。`CF_agent-gateway` main commit `f0f0ea0cbcc1029104002b566912afabd23423c7` 已实现 Message Store 来源账号隔离、`event_id` 与来源物理消息双重幂等、`is_mentioned` / `is_self`、Identity Mapping、Employee Workspace、AI Thread、Hermes Thread 绑定唯一性、Access Control 纯规则评估器、用户白名单 / 群策略 / Gateway 全局策略持久化，以及 `agent-wechat` HTTP Client、微信消息标准化、媒体 JSON / Base64 解码、文本消息真实发送字段和微信系统消息解析；当前全量 162 项测试通过。上述代码模块尚未通过 Adapter 到 Message Store 的正式接线和 Admission Orchestrator 组成运行链路；Polling / Checkpoint、Context Builder、Task Queue、Hermes 接入和端到端回传仍未实现。Gateway 已提供 Dockerfile 和 Compose 配置，但尚未完成 Docker 镜像构建和部署验证。入口已验证或模块代码已实现均不表示端到端运行或生产上线。详见[当前开发进度](./status/current-progress.md)、[agent-wechat V1 入口验证记录](./status/agent-wechat-validation.md)和[部署运维](./04_部署运维.md)。
