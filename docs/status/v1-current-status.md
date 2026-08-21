# V1 当前状态

> 文档更新日期：2026-08-20
>
> 状态证据基线：2026-08-14
>
> 文档定位：系统级状态摘要。更新文档结构不等于产生新的实机证据；能力级事实和执行顺序以[当前状态矩阵](../../status/current-status.md)为准。

## 状态口径

- **当前已完成：** 只指明确列出的范围已取得实机验证证据，不代表整个组件或总架构已经完成。
- **当前验证：** 记录已有局部证据及仍需继续验收的边界；局部通过不能外推为端到端通过。
- **当前未完成：** 尚无完整实现或验收结论，不能写成可用能力。
- **下一阶段计划：** 记录推进顺序，不表示已经开工、部署或验收。

统一结论：

> 私聊和 `group_sender` 群聊的授权文本闭环已实机验证；媒体链路、引用上下文注入和完整宿主恢复仍待完成。

## 当前已完成

| 能力 | 已验证范围 | 不包含 |
| --- | --- | --- |
| 微信文本入口 | 登录管理、手机确认登录、文本轮询、Gateway 内网通信和 Token 鉴权 | 完全新设备 SSH 二维码登录、完整媒体收发 |
| 私聊授权文本闭环 | Persist-first、身份权限、Admission、Workspace、AI Thread、路由、Hermes Dispatch、响应持久化与原会话回复 | 媒体、引用正文注入和更高恢复层级 |
| 群聊授权文本闭环 | 明确 `@` 机器人的 `group_sender` 链路已实际回复；未真实 `@` 时持久化后安全结束 | `group_shared`、群聊媒体闭环 |
| Hermes 文本执行 | 当前 Hermes 的授权文本 Dispatch 与 Response | 媒体、Skills、企业系统、多节点调度和可靠自启 |
| 文本响应与投递 | Response Persistence、Delivery Outbox、微信实际回复和 Bot 防回环 | 图片与文件回传 |
| 应用级恢复 | CFserver Gateway 应用服务 restart 后恢复持久化关系，并复用原线程上下文 | 容器 recreate、数据库、CFserver 和 AI Host 重启 |

## 当前验证

| 验证主题 | 当前已有证据 | 尚未通过的边界 |
| --- | --- | --- |
| 引用消息 | 引用类型识别、`reply_context` 持久化和引用类型消息的文本回复已验证 | 被引用正文尚未自动注入 Hermes |
| 微信图片入口 | 图片类型识别、Raw Payload、真实 JPEG 字节提取及签名、大小和 SHA-256 校验已验证 | Attachment、私有媒体存储、Hermes 多模态、READY Artifact 和微信媒体回传 |
| 身份与群聊隔离 | 测试身份的允许/拒绝、真实 mention 和 `group_sender` 隔离已验证 | 正式员工与业务群策略矩阵、`group_shared` |
| 故障与恢复 | Hermes 不可达现象和一次带证据核对、备份与 Guard 的受控人工恢复已有记录 | 正式 `uncertain` 管理、自动恢复和各级宿主重启验收 |

本节的“当前验证”表示验证边界，不表示所有列出的缺口都已进入实机测试。

## 当前未完成

| 能力 | 当前结论 |
| --- | --- |
| 媒体完整链路 | 入站 Attachment、Gateway 私有媒体存储、Hermes 多模态、出站 READY Artifact 和微信媒体投递未完成 |
| 引用上下文 | 被引用内容尚未注入 Hermes，不能声称 AI 已理解引用正文 |
| 可靠运行与恢复 | Hermes 自启、守护、健康告警、正式 `uncertain` 恢复，以及容器/数据库/宿主恢复未完成 |
| 企业文件接入 | 唯一正式 File Service、权限和审计边界已经确定，但组件仍为开发中；与 Gateway/Hermes 的稳定契约和生产端到端验收未完成 |
| Skills 与企业系统 | Skills Runtime、ERP、S6、业务数据库及其他业务自动化未进入生产验收 |
| 群共享上下文 | `group_shared` 尚未审批并实机验证 |
| 多 AI 节点 | 节点注册、调度、健康判断、容量选择、故障切换和状态一致性尚未设计或验证 |

## 下一阶段计划

以下是阶段性归类，不取代权威清单的逐项顺序：

1. **先收口单节点可靠性：** 完成 Hermes 自启、守护、告警和正式 `uncertain` 管理能力。
2. **再完成上下文与媒体：** 注入引用正文，建设入站 Attachment、Hermes 媒体协议、READY Artifact 和微信媒体回传。
3. **补齐微信入口验证：** 实机验证完全新设备 SSH 二维码登录。
4. **随后逐级验证恢复：** 分别验收容器 recreate、PostgreSQL、CFserver 和 AI Host 重启，不用较低层级结果替代较高层级。
5. **验证群共享策略：** 单独审批并实机验证 `group_shared`，不从 `group_sender` 结果外推。
6. **接入企业文件与 Skills：** 在媒体基础链稳定后接入 File Service，再建设受控 Skills Runtime。
7. **扩展业务自动化：** 按业务价值接入 ERP、S6 和其他企业系统，并分批授权正式员工与业务群。
8. **评估多 AI 节点：** 单节点权威状态和恢复边界稳定后，再形成节点调度、故障切换和容量治理的正式设计；当前不把它列为已建能力。

严格的 18 步当前执行顺序仍以[当前状态矩阵的下一阶段顺序](../../status/current-status.md#下一阶段顺序)为准；本页只做系统级归类，不复制或取代该清单。

## 相关文档

- [整体架构](../architecture/overall-architecture.md)
- [项目边界](../architecture/project-boundaries.md)
- [生产部署拓扑](../deployment/production-topology.md)
- [当前开发进度](../../status/current-progress.md)
- [2026-08-14 私聊、群聊及媒体验证记录](../../status/2026-08-14-private-group-media-validation.md)
