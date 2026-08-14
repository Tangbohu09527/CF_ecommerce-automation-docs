# 当前开发进度

> 状态日期：2026-08-14。当前能力级事实以[当前状态矩阵](./current-status.md)为准，本轮生产证据以[私聊、群聊及媒体验证记录](./2026-08-14-private-group-media-validation.md)为准。

## 当前阶段

项目仍处于阶段 1，当前结论是：

> 私聊和 group_sender 群聊的授权文本闭环已实机验证；媒体链路、引用上下文注入和完整宿主恢复仍待完成。

## 已完成

- `CF_agent-wechat` 登录管理、手机确认登录、文本读取与发送已完成实机验证；生产不使用 VNC/noVNC。
- Gateway 五服务保持 healthy；Persist-first、Checkpoint、历史基线跳过和未授权拒绝继续有效。
- 测试身份、来源映射、两级策略、Conversation/Profile 绑定、Admission Allowed 和 V2 Routing 已用于真实私聊与群聊文本验证。
- 私聊 `private_sender` 与群聊 `group_sender` 均完成 Hermes Dispatch、Response Persistence、Delivery Outbox 和微信实际回复。
- 群聊没有真实 `@` 时持久化后以 `bot_not_mentioned` 结束；机器人回复不回环。
- 同一员工 Workspace 复用，私聊与群聊 AI/Hermes Thread 隔离；CFserver Gateway 应用服务 restart 后原线程和 Hermes 上下文继续复用。
- 引用识别、`reply_context` 持久化及引用类型消息的文本回复已验证。
- 微信图片消息、Raw Payload、真实 JPEG 字节读取及签名、大小、SHA-256 校验已验证。
- Hermes 不可达故障已复现，并完成一次带备份、证据核对和 Guard 的受控人工恢复。

## 当前阻塞

- Hermes Gateway 开机自启仍不可靠，缺少正式守护、健康告警和自动恢复。
- `uncertain` Dispatch 缺少正式查询与恢复命令/API；人工直接修改数据库不能成为常规路径。
- `reply_context` 尚未注入 Hermes，不能声称 AI 已理解被引用内容。
- Attachment、Gateway 私有媒体存储、Hermes 媒体协议、READY Artifact 和微信媒体投递尚未完成。
- 完全新设备扫码、容器 recreate、PostgreSQL、CFserver、AI 主机重启恢复及 `group_shared` 尚未验证。
- FileBrowser、Skills 和业务系统接入尚未开始生产验收。

## 下一阶段

严格按[当前状态矩阵中的 18 步](./current-status.md#下一阶段顺序)执行：先处理 Hermes 可靠性与 `uncertain`，再完成引用和媒体双桥，随后逐级验证恢复，最后接入 FileBrowser、Skills、旺店通/S6 并分批授权。

## 历史记录说明

- [2026-08-13 微信运行时收口记录](./2026-08-13-wechat-runtime-closeout.md)记录入口、Checkpoint、历史基线和未授权拒绝阶段。
- [Gateway V1 Staging 微信文本闭环验证记录](./gateway-wechat-staging-validation.md)记录 2026-08-04 特定 Staging 环境的历史结果。
- [agent-wechat V1 入口验证记录](./agent-wechat-validation.md)记录较早入口能力验证。

历史记录只说明其标注日期和环境，不能覆盖 2026-08-14 当前状态。
