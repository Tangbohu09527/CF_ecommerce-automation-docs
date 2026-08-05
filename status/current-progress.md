# 当前开发进度

> 状态日期：2026-08-04。本文记录已经验证的事实、已经形成的设计基线和仍待完成的工作；“已完成”仅限所列验证、实现或文档设计范围，不等同于生产上线或完整企业业务验收。完整 V1 Staging 文本闭环证据见[Gateway 验证记录](./gateway-wechat-staging-validation.md)。

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
| 消息发送 | `POST /api/messages/send` 已按真实 `chatId + text` 契约完成微信文本回复回传 | 图片、文件或其他富媒体结果已经能够回传 |
| `CF_agent-gateway` 工程基础 | 已完成 Python + FastAPI、YAML 配置、JSON 结构化日志、SQLAlchemy engine / session、SQLite 自动建表和 PostgreSQL 配置兼容；当前 V1 Staging Worker 已在 Debian 13 运行 | 已完成生产部署、生产数据库选型或完整 Gateway |
| Message Store | 已在真实微信联调中验证未知身份消息能够成功保存 | 已完成附件主链路、生产存储和长期运行验收 |
| 微信 Polling Runtime | 已串通 `agent-wechat`、Polling、Checkpoint 和 Message Store；常驻 Worker / 串行轮询已实现并用于 V1 Staging；`bootstrap_mode=latest` 首次启动已验证建立高水位且不消费历史消息 | 已完成重启恢复、服务管理、全部历史补拉、长期稳定性或生产可靠性验收 |
| Identity Mapping | 未配置身份的消息已验证被拒绝进入执行上下文；脱敏测试来源身份到 Enterprise Identity 的授权映射已验证 | Identity Management V1 管理面、完整员工目录或多入口身份体系已完成 |
| Access Control 与 Admission | 给定 User Policy、Gateway Policy 和 `normal` 风险级别的允许路径，以及未知身份拒绝路径，已完成 Staging 实测 | 完整岗位、群、Skill、文件和高风险确认权限矩阵已验收 |
| 员工工作区与 AI Thread | 授权消息已验证创建工作区、AI Thread、来源关系和 Hermes Runtime Thread 绑定 | 群聊同群不同员工的目标线程隔离已经验收；当前 V1 存在 whole-room thread 已知实现偏差 |
| Hermes 文本集成 | Hermes API Client、消息 Dispatch、Response Relay 和 Runtime Thread Binding 已完成 V1 Staging 验证 | Context Builder、Task Queue、完整 Worker Bridge 或 Skill 执行链已完成 |
| Gateway Staging 文本闭环 | 已验证微信文本从 Polling、身份与权限准入、Employee Workspace / AI Thread、Hermes API 调用到原微信会话回复的完整链路 | 图片 / 附件 / 文件处理、企业业务自动化、生产部署或长期稳定性已完成 |
| Self message 防回环 | Polling 对 `is_self=true` 消息不进入 sink、admission 或 Hermes，并推进 Checkpoint | 所有重复投递、断线恢复和长期运行场景已经验收 |
| Gateway Docker 配置 | 已提供 Dockerfile 和 Compose 配置；本次 Staging 使用 venv 运行 | 已完成 Gateway Docker 镜像构建、容器运行或部署验证 |
| File Service V1 Beta 核心代码 | `CF_filebrowser-enterprise` 远端 `feat/v1-integration` commit `f329de2fc6e9296ca949acab4873c30a83d5f5e7`：V1 Beta 核心代码已实现并通过自动化测试。Browse / Preview / Download 三权与 Create / Modify / Delete / Replace 权限语义，以及 Upload、覆盖、重命名、移动、删除、Preview、Thumbnail、Media、Range Download、Archive Create / Extract、ACL、路径和文件系统安全均已覆盖 | 已完成 Debian 真实部署、真实客户端联调、自动化主线接入或正式生产上线 |
| File Service API Token 与 Share | 已实现 API Token hash-only、旧 Token / Key V5 migration 与失败关闭、明文仅创建响应一次性返回；Share capability 前端 UI、Browse / Preview 派生规则、Upload Share 强制 Create、密码保持（字段缺省或 `null`）、移除、替换三态、POST / PATCH legacy Token 清理及 credential 与 active hash 精确绑定均已完成回归 | 普通用户或管理员可在前端自行扩权，或 Share 凭证可跨 hash 复用 |
| File Service Persistent Audit | 已实现 Persistent Audit Store、Audit V3 migration、Pending / Finalize / Recovery、RequestID、request-scoped Recorder、degraded 状态、管理员 Audit Query、过滤、分页、用于防止查询分页状态被篡改的签名 Cursor 校验和秘密泄漏防护；已覆盖 Token、登录、用户、权限、Share、Browse、Preview、Download、Upload、Modify、Rename、Move、Delete、Archive、WebDAV 与 OnlyOffice Action | 已实现 Audit WORM、外部防篡改存档、Audit 管理前端 UI、完整生产 retention 或部署验收 |
| File Service 自动化验证 | 前端 17 个测试文件共 109 项测试、Share capability / 密码生命周期 / credential hash、API Token UI、用户三权 UI、WebDAV / OnlyOffice Audit 和既有 Audit Action 回归，以及 frontend typecheck、lint、i18n check、production build、`go vet`、`go test ./http`、`go test ./...` 均通过 | 已完成真实 Debian、WebDAV / OnlyOffice 服务或企业业务端到端验收 |

## 待完成

| 项目 | 当前边界 |
| --- | --- |
| Context Builder | 有限上下文选择和不可变快照仍待实现 |
| Task Queue | Task 持久化、排队、租约、重试和恢复仍待实现 |
| Worker Bridge 完整链路 | Gateway 到 Windows Hermes API 的 Staging 文本调用已完成；目标 Worker Bridge 的领取任务、租约、隔离工作区、文件传输和恢复仍待实施 |
| Gateway 群聊线程隔离偏差 | 目标设计为 `bot_account_id + group_chat_id + sender_id`；当前 V1 whole-room thread 行为待修复并补充同群不同员工隔离测试，不代表设计变更 |
| 非文本结果回传 | 微信文本结果回传已完成；图片、附件、文件和其他富媒体结果回传仍未完成 |
| Gateway Docker 验证 | Dockerfile 和 Compose 配置已提供；本次只验证 venv 方式，Gateway 镜像实际构建和容器运行尚未验证 |
| Hermes 员工工作台 | 按员工显示独立工作区、线程、任务历史、队列状态、Provider、模型、耗时、上下文快照和原会话来源的界面待开发 |
| Skill 体系 | 运行边界已确定，库存、订单、文件等具体 Skill 尚待定义、实现和验收 |
| ERP/S6 接口 | 旺店通 ERP、旺店通 WMS、S6 的接口、字段、数据口径、权限和数据时效待逐项验证与对接 |
| File Service 自动化对接 | FileBridge / `filebrowser-agentctl`、Gateway、Hermes 与 Skills 对稳定 File Service API 的对接待集成；受控客户端不得直接访问正式存储或自行扩大 capability |
| File Service 真实环境验收 | Debian 真实部署、V2 -> V3 Audit migration、用户 V5 migration、migration 失败关闭、备份恢复、升级回滚、Docker / Compose、Nginx、TLS、systemd、开机自启、真实 WebDAV 客户端和真实 OnlyOffice 服务仍待验证或联调 |
| File Service 发布 | V1 Beta Candidate、Git tag、GitHub Release 和正式生产上线仍待完成；当前只是核心代码冻结候选，不得表述为已正式部署 |
| 文件自动处理 | 文件消息和 ZIP 入口已验证；图片理解、附件系统级传递、临时文件管理、ZIP / 压缩包解析、OCR / 视觉、Skill 处理、产物回写、归档以及自动化主线对稳定 File Service API 的调用均待集成 |
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
- 图片理解、附件系统级传递、文件内容解析、视觉模型和企业知识库属于后续规划，不代表当前已经具备。
- 合并转发消息仅完成外层识别；内部聊天记录展开和内部文件自动提取属于待开发的增强解析能力，不影响普通微信文件入口。
- 第一阶段不建设独立 OCR；是否补充 OCR 能力需按实际业务需求重新评估。
- `CF_filebrowser-enterprise` 是正式企业 File Service；FileBridge / `filebrowser-agentctl` 只是受控客户端。正式文件自动化访问必须经过 File Service API、文件权限、Token capability、Share capability 和 Audit，不建设第二套 File Service。

## 当前阶段结论

项目仍处于阶段 1，但 V1 Staging 微信文本消息 AI 闭环已经完成：真实微信文本经过 Polling、Message Store、Identity / Permission Admission、Employee Workspace / AI Thread 后调用 Windows Hermes API，并把文本响应返回原微信会话；Hermes Client、Dispatch、Response Relay、Runtime Thread Binding 和 self message 防回环均已验证。下一步集中在修复微信群 whole-room thread 与既定 `group + sender` 隔离设计的偏差，以及 Identity Management V1、Context Builder、Task Queue、完整 Worker Bridge、Skill、ERP/S6 和文件主链路。图片理解、附件传递、文件处理、OCR、压缩包解析、企业知识库、Skill 自动执行和生产自动部署仍未完成。该结果不代表生产上线或完整企业业务自动化。完整证据见[Gateway V1 Staging 微信文本闭环验证记录](./gateway-wechat-staging-validation.md)。

`CF_filebrowser-enterprise` 是唯一正式企业 File Service；FileBridge / `filebrowser-agentctl` 只是受控客户端，Gateway、Hermes、Skills 和客户端均不得绕过 File Service API、文件权限、Token capability、Share capability 或 Persistent Audit。V1 Beta 核心代码已实现并通过自动化测试，Share capability 前端、密码三态与 hash 绑定，以及 Token、登录、用户、权限、Share、核心文件、Archive、WebDAV 和 OnlyOffice Audit Action 均已覆盖。FileBridge / Gateway / Hermes / Skills 自动化对接、Debian 与 migration 真实环境验证、备份恢复、升级回滚、真实客户端联调、V1 Beta Candidate、Git tag、GitHub Release 和正式生产上线仍未完成。Gateway 已完成的 Staging 文本闭环与 FileBrowser 已实现的模块能力都不等于文件 / Skill / 企业业务端到端运行或生产上线；在剩余环节完成并通过验收前，不对外宣称企业业务自动化已经可用。验证范围详见[Gateway V1 Staging 验证记录](./gateway-wechat-staging-validation.md)和[agent-wechat V1 入口验证记录](./agent-wechat-validation.md)，员工隔离边界见[员工工作区与 AI 会话线程设计](../design/employee-workspace-design.md)。
