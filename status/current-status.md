# 当前状态矩阵

> 状态日期：2026-08-14。本文是总体项目当前状态入口；“当前状态”只评价该行明确限定的能力范围，不能外推为整个组件或全部恢复层级均已完成。

## 状态口径

- **已部署并实机验证：** 当前部署存在，且该行明确列出的关键运行行为已在目标环境验证。
- **已部署，部分链路待验证：** 当前部署存在，但该行端到端范围或关键恢复能力尚未全部实机验证。
- **开发中：** 已有设计或实现工作，尚未达到当前目标部署与验收范围。
- **规划中：** 尚未进入当前实施与验收阶段。

## 能力状态

| 组件或能力 | 所属仓库或组件 | 运行位置 | 当前状态 | 已验证证据 | 未验证边界 | 下一动作 |
| --- | --- | --- | --- | --- | --- | --- |
| `agent-wechat` 登录与文本接口 | `CF_agent-wechat` | CFserver | **已部署并实机验证** | 容器图形环境、登录管理脚本、手机确认登录、内网 Token 鉴权、文本读取与实际回复已验证 | 完全新设备 SSH 二维码扫码；图片/文件 `send_media` | 验证完全新设备登录，并配合 Gateway 完成媒体发送 |
| 私聊 V2 Runtime | `CF_agent-gateway` | CFserver | **已部署并实机验证** | 从 Message Store、身份与两级策略、Admission Allowed、`private_sender`、Workspace、AI Thread、V2 Routing、Hermes 到微信回复全链路通过 | 媒体、引用正文注入及更高恢复层级不在本行结论内 | 保持回归验证并纳入容器与宿主恢复演练 |
| 群聊 `@` Runtime | `CF_agent-gateway` | CFserver | **已部署并实机验证** | 普通群消息未 `@` 时持久化并以 `bot_not_mentioned` 结束；真正 `@` 时 `group_sender` 完整文本闭环和实际回复通过 | `group_shared` 未实机验证；图片/文件群聊闭环未完成 | 完成 `group_shared` 审批与独立验证 |
| 身份与权限 | Gateway Identity / Access Control | CFserver | **已部署并实机验证** | 测试身份的 Enterprise Identity、Source Identity Mapping、User Policy、Gateway Policy、允许和拒绝路径通过 | 完整管理员能力、正式员工与业务群的策略矩阵未验收 | 在主链路稳定后分批授权正式员工和群 |
| Workspace / AI Thread | Gateway Conversation Runtime | CFserver + Hermes | **已部署并实机验证** | 同一员工 Workspace 复用；私聊/群聊 AI Thread 与 Hermes Thread 隔离；同群同发送者复用 `group_sender` 上下文；CFserver Gateway 应用服务 restart 后继续复用 | 容器 recreate、数据库及宿主重启后的绑定恢复未验证 | 按恢复层级逐项演练 |
| Hermes | Hermes Gateway 0.20.0 | Windows AI 主机 | **已部署，部分链路待验证** | 私聊与群聊文本 Dispatch/Response 通过；不可达故障和一次受控人工恢复已有证据 | 开机自启、守护、告警、AI 主机重启自动恢复、媒体和 Skills | 先收口开机自启、健康监控与 `uncertain` 管理 |
| 文本 Response / Delivery | Gateway Response + Delivery | CFserver | **已部署并实机验证** | Response Persistence、Delivery Outbox、`delivery-worker`、微信实际回复和 Bot 防回环通过 | 图片/文件 Artifact 与媒体投递不在本行结论内 | 建设 READY Artifact 后验证媒体投递 |
| 引用消息 | `agent-wechat` + Gateway | CFserver | **已部署，部分链路待验证** | 引用识别、`reply_context` 持久化、引用类型消息文本回复通过；群聊无真实 `@` 时安全拒绝 | 被引用内容尚未自动注入 Hermes | 将 `reply_context` 受控注入 Hermes 请求并回归测试 |
| 微信图片发现与提取 | `agent-wechat` + Gateway 消息入口 | CFserver | **已部署，部分链路待验证** | `image/raw_type=3`、Message Store、Raw Payload、真实 JPEG 字节、签名/大小/SHA-256 校验通过；无 `@` 不误触发 AI | Attachment、私有存储、Hermes 多模态未接入 | 建设入站 Attachment 与私有媒体存储 |
| 文件入站 | Gateway Media Runtime | CFserver | **开发中** | 已形成受控持久化、校验、生命周期与权限设计 | 常见办公文件、PDF、压缩包、中文文件名、大小边界和实机收取均未验证 | 实现入站文件桥并逐类实测 |
| Artifact 出站 | Gateway Artifact / Delivery | CFserver + Hermes | **开发中** | 已形成 `artifact_ref`、受认证下载、原子物化、`READY`、Outbox 与 `send_media` 设计 | ArtifactRepository、下载协议、媒体投递和微信实际接收未完成 | 实现出站桥，先验证图片再验证文件 |
| Hermes 配置档案路由 | Gateway Routing + Hermes | CFserver + Windows AI 主机 | **已部署并实机验证** | 私聊与 AI 群均通过 `external_profile_ref` 选择 `default`；会话仍保持隔离 | 多档案切换和更多能力档案未验收；档案由 Hermes 外部管理 | 增加多档案场景时单独验证绑定和权限 |
| PostgreSQL | `CF_agent-gateway` | CFserver | **已部署，部分链路待验证** | 已支撑 Message Store、Checkpoint、身份权限、线程、路由、Response 与 Outbox 的实测和 CFserver Gateway 应用服务 restart 恢复 | PostgreSQL 重启及完整恢复演练未验证 | 备份后执行数据库重启恢复验收 |
| `CF_filebrowser-enterprise` | `CF_filebrowser-enterprise` | 以该项目当前文档为准 | **开发中** | 本次不改变其既有验证结论 | 尚未接入微信媒体、Hermes 和企业资料主链路 | 媒体基础桥完成后接入 File Service 与企业资料 |
| Skills | Hermes / Windows 执行侧 | Windows AI 主机 | **规划中** | 本轮无生产实机验证 | Skills Runtime、权限、审计和业务工具未接入 | FileBrowser 接入后按业务价值实施 |
| OCR | 后续能力 | 未部署 | **规划中** | 本轮无实机验证 | 第一阶段不建设独立 OCR | 依据真实业务需求决定是否立项 |
| 电商业务自动化 | 旺店通 / S6 等业务集成 | 待定 | **规划中** | 本轮无生产实机验证 | 接口、字段、权限、幂等、确认和回滚未定义完毕 | 在消息、媒体、文件和 Skills 主链路稳定后分业务接入 |

## 当前阶段结论

> 私聊和 group_sender 群聊的授权文本闭环已实机验证；媒体链路、引用上下文注入和完整宿主恢复仍待完成。

本轮“CFserver Gateway 应用服务 restart”仅指 CFserver Gateway 应用服务重启，不包含容器 recreate、PostgreSQL、CFserver、AI 主机或 Hermes 重启；该结果不能替代更高层级验收。图片字节能够读取和校验，也不能外推为 Hermes 已能看图或微信已能收到 AI 生成图片。

## 下一阶段顺序

1. 收口 Hermes Gateway 开机自启、守护和告警。
2. 建设 `uncertain` Dispatch 正式管理与恢复命令/API。
3. 将 `reply_context` 注入 Hermes 请求。
4. 建设微信入站 Attachment 和私有媒体存储。
5. 定义并实现 Hermes 入站媒体上传/引用协议。
6. 实现 Hermes 出站 Artifact 下载和物化。
7. 实机验证微信图片发送。
8. 实机验证微信文件收发。
9. 实机验证完全新设备 SSH 二维码登录。
10. 验证 Gateway 容器强制重建恢复。
11. 验证 PostgreSQL 重启恢复。
12. 验证 CFserver 整机重启恢复。
13. 验证 AI 主机重启和 Hermes 自动恢复。
14. 验证 `group_shared` 策略。
15. 接入 FileBrowser 企业文件中心。
16. 建设 Skills Runtime。
17. 接入旺店通/S6 业务流程。
18. 分批授权正式员工和业务群。
