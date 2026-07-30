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
- **已验证：** `agent-wechat` V1 入口已完成微信登录、私聊文本、群聊文本、文件消息、ZIP 文件、引用消息、`sender` 识别和 `chatId` 识别验证。
- **待技术验证：** 图片、Office、PDF、中文文件名、连续多附件、失败重试和长期运行稳定性等未覆盖场景。

## 文档入口

| 文档 | 内容 |
| --- | --- |
| [项目总纲](./00_项目总纲.md) | 范围、总体模块、设备分工、建设阶段 |
| [功能需求](./01_功能需求.md) | 系统要做什么及验收边界 |
| [系统设计](./02_系统设计.md) | 架构、职责、数据流、状态与安全边界 |
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
| `filebrowser-enterprise` | **已确定：** 当前继续并行二次开发，暂不重命名；本仓库任务不得修改它 |
| 其他 `CF_` 前缀代码仓库 | **后续规划：** 尚未创建，名称与边界须另行确认 |

## 当前状态

当前仍处于**阶段 1**。Windows AI 节点与 Hyper-V Debian 测试节点的基础运行环境已经完成，`agent-wechat` V1 入口验证已经完成：微信登录、私聊/群聊文本、文件消息、ZIP 文件、引用消息以及 `sender`、`chatId` 识别均已验证。微信消息入口层技术可行，但该结论不代表生产上线或端到端主链路完成；Gateway、Hermes Agent、Skills、ERP/S6 接口、正式文件处理、权限体系和 WebSocket 实时事件仍待设计、开发、接入或研究。详见[当前开发进度](./status/current-progress.md)、[agent-wechat V1 入口验证记录](./status/agent-wechat-validation.md)和[部署运维](./04_部署运维.md)。
