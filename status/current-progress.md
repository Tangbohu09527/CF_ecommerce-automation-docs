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
| `CF_agent-gateway` 工程基础 | 已完成 Python 3.12 + FastAPI、YAML 配置、JSON 结构化日志、SQLAlchemy engine / session、SQLite 自动建表和 PostgreSQL 配置兼容；commit `587f59f` 已使用 Python 3.13.5 完成 Debian Staging venv 部署验证 | 已完成生产部署、生产数据库选型或完整 Gateway |
| Message Store | commit `587f59f` 已在真实微信联调中验证未知身份消息能够成功保存 | 已完成附件主链路、生产存储和长期运行验收 |
| 微信 Polling Runtime | commit `587f59f feat: wire wechat polling runtime` 已串通 `agent-wechat`、Polling、Checkpoint 和 Message Store；`bootstrap_mode=latest` 首次启动已验证建立高水位且不消费历史消息 | 已完成常驻调度、全部历史补拉、长期稳定性或生产验收 |
| Identity Mapping | 未配置身份的消息已验证被拒绝进入执行上下文；测试微信 ID 到 `EMP_TEST_001` 的授权映射已验证 | Identity Management V1 管理面、完整员工目录或多入口身份体系已完成 |
| Access Control 与 Admission | 给定 User Policy、Gateway Policy 和 `normal` 风险级别的允许路径，以及未知身份拒绝路径，已完成 Staging 实测 | 完整岗位、群、Skill、文件和高风险确认权限矩阵已验收 |
| 员工工作区与 AI Thread | 授权消息已验证创建 `employee_workspaces`、`ai_threads` 和 `thread_source_bindings` | Hermes 已接入、`hermes_thread_id` 已创建、运行时线程恢复或员工工作台已完成 |
| Gateway Staging 真实联调 | commit `587f59f` 已验证真实微信入口到 AI Thread 的权限执行链 | 生产上线、完整 AI Agent 闭环、Hermes 运行、AI 回复微信或 Skill 执行已完成 |
| Gateway Docker 配置 | 已提供 Dockerfile 和 Compose 配置；本次 Staging 使用 venv 运行 | 已完成 Gateway Docker 镜像构建、容器运行或部署验证 |
| File Service 安全与 capability | `CF_filebrowser-enterprise` 远端 `feat/v1-integration` commit `4096a0161952a5faa43058f251f8dff0dcedf890` 已实现 API Token hash-only 存储、旧 Token/Key V5 migration 与 mandatory migration fail-closed、Token 前端一次性展示（明文只在创建响应返回）、Browse / Preview / Download 三权 UI、Archive / Extract 权限与文件系统安全、Share `configured` / `effective` capability 服务端契约和密码 Share Token 绕过关闭 | Share capability 前端 UI、自动化主线对接、最终 Debian 部署验收、正式生产上线或 V1 Beta 最终 tag/candidate 已完成 |
| File Service Audit 基础 | 同一集成基线已实现 Persistent Audit Store、Audit V3 migration、request-scoped Audit Recorder、API Token 创建 / 撤销 / 拒绝 Audit、管理员 `GET /api/audit`、Audit 过滤、Cursor 和日志脱敏 | 登录、用户、权限、Share 管理、核心文件、Archive、WebDAV、OnlyOffice 等全部业务 Audit Action 已完成 |

## 待完成

| 项目 | 当前边界 |
| --- | --- |
| Context Builder | 有限上下文选择和不可变快照仍待实现 |
| Task Queue | Task 持久化、排队、租约、重试和恢复仍待实现 |
| Gateway Hermes Adapter | AI Thread 已能创建，但 Hermes Runtime 尚未接入，当前 `hermes_thread_id` 为空 |
| Hermes 与 Worker Bridge | 生产 Agent 路线已确定；Hermes、Worker Bridge、GPT-5.6 API 调用和任务链路尚待实施 |
| AI 回复回传微信 | 文本消息真实发送字段已有代码实现，但 AI 结果经 Gateway 返回原微信会话的完整链路尚未实现 |
| Gateway Docker 验证 | Dockerfile 和 Compose 配置已提供；本次只验证 venv 方式，Gateway 镜像实际构建和容器运行尚未验证 |
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
| 企业权限体系剩余项 | 给定 User Policy、Gateway Policy 和 `normal` 风险级别的允许 / 拒绝路径已完成 Staging 实测；岗位、群、Skill、系统接口、文件路径、管理员跨员工查看权限矩阵、高风险确认、完整发布与审计闭环仍待业务确认和实施 |

## Roadmap

### Identity Management V1

- 管理员、员工角色。
- 管理员新增员工。
- 管理员移除员工。
- 白名单管理。
- 管理员查看微信昵称、微信备注、微信头像和最近活跃时间。

微信 ID 作为微信来源身份的稳定主键和映射键；Gateway 内部身份、工作区和权限关联的权威主键仍为 `enterprise_identity_id`。微信昵称和备注仅用于管理员展示，不用于授权或自动合并身份。完整需求见[功能需求](../01_功能需求.md#11-identity-management-v1)。

## 规划能力说明

- 微信结构化 mention 已验证并固化为 `is_mentioned = raw.get("isMentioned") is True`；字段缺失按 `false`，不得从正文、机器人当前或旧名称、引用消息或上一条 mention 推断。
- 图片、Office、PDF、中文文件名、连续多附件、大小边界和失败重试仍需 `agent-wechat` 技术验证；ZIP 文件入口已经验证。
- 文件内容解析、视觉模型和知识库属于后续规划，不代表当前已经具备。
- 合并转发消息仅完成外层识别；内部聊天记录展开和内部文件自动提取属于待开发的增强解析能力，不影响普通微信文件入口。
- 第一阶段不建设独立 OCR；是否补充 OCR 能力需按实际业务需求重新评估。
- `CF_filebrowser-enterprise` 是正式企业 File Service；FileBridge / `filebrowser-agentctl` 只是受控客户端。正式文件自动化访问必须经过 File Service API、文件权限、Token capability、Share capability 和 Audit，不建设第二套 File Service。

## 当前阶段结论

项目仍处于阶段 1。`agent-wechat` V1 微信入口层和结构化 mention 已在限定范围验证，合并转发当前只完成外层识别。`CF_agent-gateway` commit `587f59f` 已在 Debian Staging 完成真实微信消息从 Gateway Runtime、Polling / Checkpoint、Message Store、Identity Mapping、Access Control 和 Admission 到 Employee Workspace / AI Thread 的联调验证。下一步集中在 Identity Management V1、Context Builder、Task Queue、Gateway Hermes Adapter、Hermes / Worker Bridge、AI 回复回传微信和 Skill 执行链，并继续建设 ERP/S6 接口及文件主链路；Gateway Docker、实时事件机制与合并转发解析增强仍待验证或开发。该结果不代表生产上线或完整 AI Agent 闭环。验证证据见[Gateway Debian Staging 真实微信联调验证记录](./gateway-wechat-staging-validation.md)，入口范围见[agent-wechat V1 入口验证记录](./agent-wechat-validation.md)，员工隔离边界见[员工工作区与 AI 会话线程设计](../design/employee-workspace-design.md)。

`CF_filebrowser-enterprise` 是唯一正式企业 File Service；FileBridge / `filebrowser-agentctl` 只是受控客户端，Gateway、Hermes、Skills 和客户端均不得绕过 File Service API、文件权限、Token capability、Share capability 或 Audit。V1 Beta 核心服务端能力已经实现，但 Share capability 前端 UI、登录 / 用户 / 权限 / Share 管理 Audit、核心文件与 Archive Audit Action、WebDAV / OnlyOffice Audit Action、FileBridge / Gateway / Hermes 自动化对接、最终 Debian 部署验收和 V1 Beta 最终 tag/candidate 仍未完成，因此不得表述为正式生产上线或完整 Audit 业务覆盖。Gateway 已完成的 Staging 入站权限链与 FileBrowser 已实现的模块能力都不等于完整端到端运行或生产上线；在剩余环节完成并通过验收前，不对外宣称企业业务自动化已经可用。验证范围详见[Gateway Debian Staging 真实微信联调验证记录](./gateway-wechat-staging-validation.md)和[agent-wechat V1 入口验证记录](./agent-wechat-validation.md)，员工隔离边界见[员工工作区与 AI 会话线程设计](../design/employee-workspace-design.md)。
