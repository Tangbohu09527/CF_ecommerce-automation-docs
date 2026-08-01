# 当前开发进度

> 状态日期：2026-08-01。本文记录已经验证的事实、已经形成的设计基线和仍待完成的工作；“已完成”仅限所列验证或文档设计范围，不等同于生产上线或端到端业务验收。

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
| 合并转发消息 | 已验证类型识别、发送人获取和外层标题获取 | 已支持展开内部聊天记录或自动提取内部文件 |
| 消息读取方式 | API 读取方式已验证 | WebSocket 实时事件已经研究或实现 |
| 消息发送 | 已验证可发送微信消息 | Gateway、Hermes 处理结果已经能够端到端回传 |
| `CF_agent-gateway` 工程基础 | 已完成 Python 3.12 + FastAPI、YAML 配置、JSON 结构化日志、SQLAlchemy engine / session、SQLite 自动建表和 PostgreSQL 配置兼容 | 已完成生产部署、生产数据库选型或完整 Gateway |
| Message Store Foundation | commit `d32b65aa389626f820349367d2132b7d53d0ed4f` 已实现 Conversation / Message / Attachment、`POST /internal/messages`、`GET /messages/{id}`、`GET /conversations/{conversation_id}/messages` 和 `event_id` 幂等；9 项测试通过 | Identity Mapping、权限、上下文、Task 或 Adapter / Hermes 链路已经实现 |
| Gateway Docker 配置 | 已提供 Dockerfile 和 Compose 配置 | 已完成 Docker 镜像构建、容器运行或部署验证 |
| 员工工作区与 AI Thread 设计 | 已完成 Enterprise Identity / 企业身份、Employee Workspace / 员工工作区、AI Thread / AI 会话线程及 Hermes Runtime Thread / Hermes 运行时线程的设计基线 | Identity Mapping、Employee Conversation Manager、Hermes 员工工作台、工作区恢复或端到端接入已经实现 |

## 待完成

| 项目 | 当前边界 |
| --- | --- |
| Hermes 接入 | 生产 Agent 路线已确定，Hermes Adapter、Worker Bridge、GPT-5.6 API 调用和任务链路尚待实施 |
| Gateway 剩余实现 | Gateway 架构、Message Store、Access Control、Task Queue 和 Hermes 事件协议已形成设计基线；工程基础和 Message Store Foundation 已实现，Identity Mapping、Employee Conversation Manager、Access Control、Context Builder、Task Queue、Adapter、Hermes 与 AI Provider 链路仍待实施 |
| Gateway Docker 验证 | Dockerfile 和 Compose 配置已提供；镜像实际构建、容器运行和部署验证尚未完成 |
| Identity Mapping 与 Employee Conversation Manager | 来源账号到 Enterprise Identity / 企业身份的映射、Employee Workspace / 员工工作区实体、AI Thread / AI 会话线程绑定和恢复待开发 |
| Hermes 员工工作台 | 按员工显示独立工作区、线程、任务历史、队列状态、Provider、模型、耗时、上下文快照和原会话来源的界面待开发 |
| Skill 体系 | 运行边界已确定，库存、订单、文件等具体 Skill 尚待定义、实现和验收 |
| ERP/S6 接口 | 旺店通 ERP、旺店通 WMS、S6 的接口、字段、数据口径、权限和数据时效待逐项验证与对接 |
| 文件自动处理 | 文件消息和 ZIP 入口已验证；临时文件管理、自动解压、OCR/视觉、Skill 处理、File Service 对接、产物回写和归档均待完成 |
| 实时事件机制 | 当前 API 读取方式已验证；WebSocket 接口、事件范围、重连、去重和补偿机制待研究和开发 |
| 合并转发解析增强 | 内部聊天记录展开、内部文件自动提取和 `forward parser` 尚待开发 |
| 企业权限体系 | Access Control 已形成设计基线；用户、群、岗位、Skill、系统接口、文件路径、管理员跨员工查看权限矩阵及高风险确认机制待业务确认和实施 |
| 端到端主链路 | 微信事件到 Gateway、Hermes、Skill、企业系统再返回原会话的完整链路尚未完成 |

## 规划能力说明

- 图片、Office、PDF、中文文件名、连续多附件、大小边界和失败重试仍需 `agent-wechat` 技术验证；ZIP 文件入口已经验证。
- 文件内容解析、视觉模型和知识库属于后续规划，不代表当前已经具备。
- 合并转发消息仅完成外层识别；内部聊天记录展开和内部文件自动提取属于待开发的增强解析能力，不影响普通微信文件入口。
- 第一阶段不建设独立 OCR；是否补充 OCR 能力需按实际业务需求重新评估。
- FileBrowser Enterprise 与自动化主线并行推进，正式文件自动化访问仍必须经过 File Service、权限检查和审计。

## 当前阶段结论

项目仍处于阶段 1。`agent-wechat` V1 微信入口层已经验证，说明微信消息入口层技术可行；合并转发当前只完成外层识别。Gateway 架构、Message Store、Access Control、Task Queue、Hermes 事件协议和员工工作区与 AI Thread 已形成设计基线，`CF_agent-gateway` 工程基础和 Message Store Foundation 已完成代码实现。下一步工作集中在 Identity Mapping、Employee Conversation Manager、Access Control、Context Builder、Task Queue、Adapter 与 Hermes / AI Provider 链路、Hermes 员工工作台、Skills、ERP/S6 接口和端到端主链路建设；Docker 实际构建部署、实时事件机制与合并转发解析增强仍待验证或开发。在这些环节完成并通过验收前，不对外宣称企业业务自动化已经可用。验证范围详见[agent-wechat V1 入口验证记录](./agent-wechat-validation.md)，员工隔离边界见[员工工作区与 AI 会话线程设计](../design/employee-workspace-design.md)。
