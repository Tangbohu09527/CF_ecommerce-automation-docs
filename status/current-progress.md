# 当前开发进度

> 状态日期：2026-08-03。本文记录已经验证的事实、已经形成的设计基线和仍待完成的工作；“已完成”仅限所列验证、实现或文档设计范围，不等同于生产上线或端到端业务验收。

## 已完成

| 项目 | 已完成 / 已验证范围 | 不代表 |
| --- | --- | --- |
| `agent-wechat` 部署验证 | 已完成测试环境部署验证 | 已形成生产高可用部署 |
| 微信登录验证 | 已完成微信登录验证 | 登录态备份、自动恢复和长期稳定性已经验收 |
| 联系人读取 | 已验证可读取联系人信息 | 已完成企业身份、岗位和权限映射 |
| 聊天读取 | 已验证可读取聊天信息 | 已完成 AI 线程隔离、上下文筛选和快照 |
| 文本与引用消息 | 已验证私聊文本、群聊文本和引用消息 | 所有消息类型、重复事件和持续运行场景已全部验收 |
| 文件入口 | 已验证文件消息和 ZIP 文件 | 已完成自动解压、内容解析、正式归档或所有附件格式兼容性验证 |
| 文件获取 | 已验证可获取微信文件 | 已完成文件内容处理、权限检查或正式归档 |
| 入口标识 | 已验证 `sender` 和 `chatId` 识别 | 已完成企业身份授权、AI 线程映射或任务权限校验 |
| 结构化 mention | 已验证成员列表选择当前机器人时 `isMentioned=true`，选择其他成员或只输入机器人名称时字段缺失 | 可根据正文 `@`、名称、引用或历史 mention 推断 |
| 合并转发消息 | 已验证类型识别、发送人获取和外层标题获取 | 已支持展开内部聊天记录或自动提取内部文件 |
| 消息读取方式 | API 读取方式已验证 | WebSocket 实时事件已经研究或实现 |
| 消息发送 | 已验证可发送微信消息 | Gateway、Hermes 处理结果已经能够端到端回传 |
| `CF_agent-gateway` 工程基础 | 已完成 Python 3.12 + FastAPI、YAML 配置、JSON 结构化日志、SQLAlchemy engine / session、SQLite 自动建表和 PostgreSQL 配置兼容 | 已完成生产部署、生产数据库选型或完整 Gateway |
| Message Store | main commit `f0f0ea0cbcc1029104002b566912afabd23423c7` 已实现 Conversation / Message / Attachment、来源账号隔离、`event_id` 与来源物理消息双重幂等 | Adapter 已正式写入、Polling / Checkpoint 或后续准入链路已运行 |
| 微信适配基础 | 已实现 `agent-wechat` HTTP Client、微信消息标准化、`is_mentioned` / `is_self`、媒体 JSON / Base64 解码、文本消息真实发送字段和微信系统消息解析 | Adapter 到 Message Store 已接线、附件或结果已端到端流转 |
| Identity Mapping | 已实现来源身份映射代码 | 已通过 Admission Orchestrator 进入正式消息准入链路 |
| 员工工作区与 AI Thread | 已实现 Employee Workspace、AI Thread 和 Hermes Thread 绑定唯一性 | Hermes 已接入、运行时线程恢复或员工工作台已完成 |
| Access Control | 已实现纯规则评估器 | 已形成从消息到 Task 的正式准入调用链 |
| 权限策略持久化 | 已实现用户白名单、群策略和 Gateway 全局策略持久化 | Skill 权限、完整策略发布、管理面、审批和审计闭环已完成 |
| Gateway 全量测试 | main commit `f0f0ea0cbcc1029104002b566912afabd23423c7` 全量 162 项测试通过 | 端到端、部署或生产验收已经通过 |
| Gateway Docker 配置 | 已提供 Dockerfile 和 Compose 配置 | 已完成 Docker 镜像构建、容器运行或部署验证 |
| File Service 安全与 capability | `CF_filebrowser-enterprise` 远端 `feat/v1-integration` commit `4096a0161952a5faa43058f251f8dff0dcedf890` 已实现 API Token hash-only 存储、旧 Token/Key V5 migration 与 mandatory migration fail-closed、Token 前端一次性展示（明文只在创建响应返回）、Browse / Preview / Download 三权 UI、Archive / Extract 权限与文件系统安全、Share `configured` / `effective` capability 服务端契约和密码 Share Token 绕过关闭 | Share capability 前端 UI、自动化主线对接、最终 Debian 部署验收、正式生产上线或 V1 Beta 最终 tag/candidate 已完成 |
| File Service Audit 基础 | 同一集成基线已实现 Persistent Audit Store、Audit V3 migration、request-scoped Audit Recorder、API Token 创建 / 撤销 / 拒绝 Audit、管理员 `GET /api/audit`、Audit 过滤、Cursor 和日志脱敏 | 登录、用户、权限、Share 管理、核心文件、Archive、WebDAV、OnlyOffice 等全部业务 Audit Action 已完成 |

## 待完成

| 项目 | 当前边界 |
| --- | --- |
| Adapter 到 Message Store 正式接线 | HTTP Client、标准化与 Message Store 已分别实现基础代码，但正式写入链路尚未完成 |
| Polling / Checkpoint | V1 目标方案已设计，轮询同步与按来源账号 / 会话维护检查点尚未实现 |
| Admission Orchestrator | Message Store、Identity Mapping、Access Control 与工作区 / 线程能力尚未被正式编排为消息准入链路 |
| Context Builder | 有限上下文选择和不可变快照仍待实现 |
| Task Queue | Task 持久化、排队、租约、重试和恢复仍待实现 |
| Hermes 接入 | 生产 Agent 路线已确定，Hermes、Worker Bridge、GPT-5.6 API 调用和任务链路尚待实施 |
| 端到端回传 | 文本消息真实发送字段已有代码实现，但 Gateway 结果经 Adapter 返回原微信会话的完整链路尚未实现 |
| Gateway Docker 验证 | Dockerfile 和 Compose 配置已提供；镜像实际构建、容器运行和部署验证尚未完成 |
| Hermes 员工工作台 | 按员工显示独立工作区、线程、任务历史、队列状态、Provider、模型、耗时、上下文快照和原会话来源的界面待开发 |
| Skill 体系 | 运行边界已确定，库存、订单、文件等具体 Skill 尚待定义、实现和验收 |
| ERP/S6 接口 | 旺店通 ERP、旺店通 WMS、S6 的接口、字段、数据口径、权限和数据时效待逐项验证与对接 |
| Share capability 前端 UI | 服务端 `configured` / `effective` capability 契约已实现，Share capability 前端 UI 仍待集成 |
| File Service 业务 Audit Action | 登录、用户、权限、Share 管理 Audit，核心文件和 Archive Audit Action，以及 WebDAV / OnlyOffice Audit Action 仍待集成；Audit Store / Recorder 已实现不代表这些 Action 已覆盖 |
| File Service 自动化对接 | FileBridge / `filebrowser-agentctl`、Gateway 与 Hermes 对稳定 File Service API 的对接待集成；受控客户端不得直接访问正式存储或自行扩大 capability |
| File Service 部署与发布 | 最终 Debian 部署验收和 V1 Beta 最终 tag/candidate 待验证；当前尚未正式生产部署 |
| 文件自动处理 | 文件消息和 ZIP 入口已验证；临时文件管理、自动解压、OCR/视觉、Skill 处理、产物回写、归档以及自动化主线对稳定 File Service API 的调用均待集成 |
| 实时事件机制 | 当前 API 读取方式已验证；WebSocket 接口、事件范围、重连、去重和补偿机制待研究和开发 |
| 合并转发解析增强 | 内部聊天记录展开、内部文件自动提取和 `forward parser` 尚待开发 |
| 企业权限体系剩余项 | Access Control 纯规则评估器与三类策略持久化已实现；岗位、Skill、系统接口、文件路径、管理员跨员工查看权限矩阵、高风险确认、完整发布与审计闭环仍待业务确认和实施 |

## 规划能力说明

- 微信结构化 mention 已验证并固化为 `is_mentioned = raw.get("isMentioned") is True`；字段缺失按 `false`，不得从正文、机器人当前或旧名称、引用消息或上一条 mention 推断。
- 图片、Office、PDF、中文文件名、连续多附件、大小边界和失败重试仍需 `agent-wechat` 技术验证；ZIP 文件入口已经验证。
- 文件内容解析、视觉模型和知识库属于后续规划，不代表当前已经具备。
- 合并转发消息仅完成外层识别；内部聊天记录展开和内部文件自动提取属于待开发的增强解析能力，不影响普通微信文件入口。
- 第一阶段不建设独立 OCR；是否补充 OCR 能力需按实际业务需求重新评估。
- `CF_filebrowser-enterprise` 是正式企业 File Service；FileBridge / `filebrowser-agentctl` 只是受控客户端。正式文件自动化访问必须经过 File Service API、文件权限、Token capability、Share capability 和 Audit，不建设第二套 File Service。

## 当前阶段结论

项目仍处于阶段 1。`agent-wechat` V1 微信入口层和结构化 mention 已在限定范围验证，说明微信入口技术可行；合并转发当前只完成外层识别。`CF_agent-gateway` main commit `f0f0ea0cbcc1029104002b566912afabd23423c7` 已实现 Message Store、身份、工作区 / 线程、Access Control、策略持久化与微信适配等基础代码，全量 162 项测试通过。下一步集中在 Adapter 到 Message Store 正式接线、Polling / Checkpoint、Admission Orchestrator、Context Builder、Task Queue、Hermes / Worker Bridge 和端到端回传，并继续建设 Skills、ERP/S6 接口及文件主链路；Docker 实际构建部署、实时事件机制与合并转发解析增强仍待验证或开发。模块代码已实现不等于正式接线、端到端运行或生产上线；在这些环节完成并通过验收前，不对外宣称企业业务自动化已经可用。验证范围详见[agent-wechat V1 入口验证记录](./agent-wechat-validation.md)，员工隔离边界见[员工工作区与 AI 会话线程设计](../design/employee-workspace-design.md)。

`CF_filebrowser-enterprise` 是唯一正式企业 File Service；FileBridge / `filebrowser-agentctl` 只是受控客户端，Gateway、Hermes、Skills 和客户端均不得绕过 File Service API、文件权限、Token capability、Share capability 或 Audit。V1 Beta 核心服务端能力已经实现，但 Share capability 前端 UI、登录 / 用户 / 权限 / Share 管理 Audit、核心文件与 Archive Audit Action、WebDAV / OnlyOffice Audit Action、FileBridge / Gateway / Hermes 自动化对接、最终 Debian 部署验收和 V1 Beta 最终 tag/candidate 仍未完成，因此不得表述为正式生产上线或完整 Audit 业务覆盖。Gateway 和 FileBrowser 的模块代码已实现均不等于正式接线、端到端运行或生产上线；在这些环节完成并通过验收前，不对外宣称企业业务自动化已经可用。验证范围详见[agent-wechat V1 入口验证记录](./agent-wechat-validation.md)，员工隔离边界见[员工工作区与 AI 会话线程设计](../design/employee-workspace-design.md)。
