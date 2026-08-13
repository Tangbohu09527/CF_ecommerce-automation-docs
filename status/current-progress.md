# 当前开发进度

> 状态日期：2026-08-13。当前组件级事实以[当前状态矩阵](./current-status.md)为准，当天生产证据以[微信运行时阶段收口记录](./2026-08-13-wechat-runtime-closeout.md)为准。

## 当前阶段

项目仍处于阶段 1，已到“**微信入口和未授权安全链路完成，授权 AI 闭环待验证**”。

> 微信消息发现、持久化、Checkpoint、未授权拒绝和 Hermes 网络连通已实机验证；授权后的完整 AI 回复闭环仍待验证。

## 已完成

- `CF_agent-wechat` 已部署在 CFserver，登录管理脚本与手机确认登录通过实机验证。
- 生产配置 `ENABLE_VNC=0`，不使用 VNC、noVNC、x11vnc、websockify 或宿主桌面 X11。
- PostgreSQL、`gateway`、`wechat-worker`、`dispatch-worker`、`delivery-worker` 五个服务已部署并保持 healthy。
- Gateway 与 `agent-wechat` 的 `cf-internal` 网络和 Token 鉴权已验证。
- 3 秒轮询、17 个 Checkpoint、151 条历史消息基线跳过、新私聊持久化和 Checkpoint 推进已验证。
- 未授权账号安全拒绝，不调用 Hermes、不产生机器人回复。
- CFserver 与 `dispatch-worker` 到 Hermes Gateway 0.20.0 的网络连通已验证。

## 当前阻塞

- Enterprise Identity、Source Identity Mapping 和两级访问策略尚未为测试发送者配置。
- Agent Profile 与私聊 Conversation-AgentProfile Binding 尚未完成。
- Admission Allowed、V2 Routing、Hermes 实际处理、Response Persistence、Delivery Outbox 与微信真实回复尚未验证。
- Hermes Gateway 在 Windows AI 主机重启后没有可靠自动启动。
- 完全新设备 SSH 二维码扫码、群聊 `@` 机器人、图片、文件和引用消息尚待生产实测。

## 下一阶段

严格按[当前状态矩阵中的 18 步](./current-status.md#下一阶段顺序)执行。完成微信真实回复验证后，再收口 Hermes 自启、群聊、附件、FileBrowser、企业资料、Skills 和业务自动化。

## 历史记录说明

- [Gateway V1 Staging 微信文本闭环验证记录](./gateway-wechat-staging-validation.md)记录 2026-08-04 特定 Staging 环境的历史结果。
- [agent-wechat V1 入口验证记录](./agent-wechat-validation.md)记录较早入口能力验证。

这些历史记录不覆盖 2026-08-13 的 CFserver 当前生产状态，也不能用于宣称当前授权后的完整 AI 回复闭环已验证。
