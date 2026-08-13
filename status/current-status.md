# 当前状态矩阵

> 状态日期：2026-08-13。本文是总体项目当前状态入口；较早的 Staging 验证记录只代表其标注日期与环境，不覆盖本文的 CFserver 生产状态。

## 状态口径

- **已部署并实机验证：** 当前部署存在，且本文明确列出的关键运行行为已在目标环境验证。
- **已部署，部分链路待验证：** 当前部署存在，但端到端链路或关键恢复能力尚未全部实机验证。
- **开发中：** 已有设计或实现工作，尚未达到当前目标部署与验收范围。
- **规划中：** 尚未进入当前实施与验收阶段。

## 组件状态

| 组件 | 职责 | 运行位置 | 当前部署状态 | 当前实机验证状态 | 当前限制 | 下一步 |
| --- | --- | --- | --- | --- | --- | --- |
| `CF_agent-wechat` | 企业 AI 微信客户端、登录状态、消息读取与发送，并向 Gateway 提供接口 | CFserver | **已部署并实机验证** | 容器内图形环境、登录管理脚本、手机确认登录、消息接口与 Gateway 内网访问已验证 | `ENABLE_VNC=0`；不使用 VNC/noVNC/x11vnc/websockify 或宿主桌面 X11；完全新设备 SSH 二维码扫码尚未实机验证 | 在受控窗口补做完全新设备扫码验证，并继续验证图片、文件和引用消息 |
| `CF_agent-gateway` | 微信轮询、Checkpoint、Message Store、身份与权限准入、Workspace、AI Thread、V2 Routing、Hermes 派发、响应持久化和投递 | CFserver | **已部署，部分链路待验证** | `gateway`、`wechat-worker`、`dispatch-worker`、`delivery-worker` 与 PostgreSQL 均保持 healthy；3 秒轮询、17 个 Checkpoint、151 条历史消息基线跳过、新私聊持久化、发送者/会话识别、Checkpoint 推进和未授权安全拒绝已验证 | 测试发送者的身份、策略、Agent Profile 与会话绑定尚未配置；授权后的 Hermes、响应持久化、Outbox 和微信回复未验证 | 按本文“下一阶段顺序”完成身份、策略、Profile、授权与完整回复闭环验证 |
| Hermes | Agent 执行、模型调用及后续 Skills/工具调用 | Windows AI 主机 | **已部署，部分链路待验证** | Hermes Gateway 0.20.0 可由 CFserver 与 `dispatch-worker` 访问；人工启动后的网络连通已验证 | Windows 登录启动项存在，但 AI 主机重启后未可靠自动启动；本轮未完成真实授权消息处理 | 完成授权消息 Dispatch 验证后，收口并复验开机自启 |
| PostgreSQL | Gateway 权威消息、Checkpoint、身份、权限、路由、响应和 Outbox 状态 | CFserver | **已部署并实机验证** | 服务 healthy；已支撑 Message Store、Checkpoint 和拒绝链路的生产实测 | 配置变更与升级前仍必须备份并验证恢复路径 | 为身份、策略、Profile 与会话绑定写入受控配置，并保留变更审计 |
| `CF_filebrowser-enterprise` | 企业文件中心、用户权限、Token、分享、WebDAV、审计日志与 AI 文件访问基础设施 | 以该项目自己的当前文档为准 | **开发中** | 本次不变更其既有验证结论 | 尚未接入本轮微信授权 AI 闭环 | 在微信文本闭环通过后接入 File Service 与企业资料 |
| Skills | 文档、浏览器、Windows 及业务系统的受控执行能力 | 计划运行于 Windows AI 主机 | **规划中** | 本轮无生产实机验证 | 尚未接入 Gateway/Hermes 生产任务链路 | FileBrowser 接入后按业务价值逐项实现和授权 |
| OCR | 图片或扫描件文字识别 | 未部署 | **规划中** | 本轮无实机验证 | 第一阶段不建设独立 OCR | 依据真实业务需求决定是否纳入后续能力 |
| 旺店通 / S6 集成 | 库存、订单、物流、财务、线下业务与对账 | 待定 | **规划中** | 本轮无生产实机验证 | 接口、权限、字段、幂等和回滚边界尚待定义 | 在消息、文件与 Skills 主链路稳定后分业务立项 |

## 当前阶段结论

当前已到“**微信入口和未授权安全链路完成，授权 AI 闭环待验证**”。准确表述为：

> 微信消息发现、持久化、Checkpoint、未授权拒绝和 Hermes 网络连通已实机验证；授权后的完整 AI 回复闭环仍待验证。

## 下一阶段顺序

1. 为测试微信发送者建立 Enterprise Identity。
2. 建立 Source Identity Mapping。
3. 配置 User Access Policy。
4. 配置 Gateway Access Policy。
5. 创建 Agent Profile。
6. 绑定私聊 Conversation-AgentProfile。
7. 发送第二条测试消息。
8. 验证 Admission Allowed。
9. 验证 V2 Routing。
10. 验证 Hermes Dispatch。
11. 验证 Response Persistence。
12. 验证 Delivery Outbox。
13. 验证微信收到真实回复。
14. 收口 Hermes 开机自启。
15. 测试群聊 `@` 机器人。
16. 测试图片、文件和引用消息。
17. 接入 FileBrowser 和企业资料。
18. 接入 Skills 和业务自动化。
