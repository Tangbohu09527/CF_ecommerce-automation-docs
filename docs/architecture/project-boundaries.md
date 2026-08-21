# 核心项目与组件职责边界

> 文档更新日期：2026-08-20
>
> 状态证据基线：2026-08-14
>
> 文档定位：统一定义微信入口、Gateway 控制面、Hermes 执行环境和企业文件系统的职责，避免把调用关系误写成职责归属或完成状态。

## 边界总览

| 项目或组件 | 核心定位 | 负责 | 明确不负责 |
| --- | --- | --- | --- |
| `CF_agent-wechat` | 微信通道入口 | 微信登录、消息读取与回复发送适配 | 企业身份权限、任务分发、AI 执行、正式文件管理 |
| `CF_agent-gateway` | 企业控制面的消息入口 | 消息接入、任务分发、上下文与全生命周期管理 | AI/Agent 实际执行、Hermes 内部配置、正式企业文件存储 |
| Hermes | AI 执行环境 | Agent 与模型执行、Hermes 配置档案、后续获准 Skills 执行 | 企业消息准入、权威任务状态、微信投递编排、绕过 File Service 访问文件 |
| `CF_filebrowser-enterprise` | 企业文件系统 / 唯一正式 File Service | 文件管理、文件权限、受控访问与持久审计 | 微信收发、AI 推理、Gateway 消息和任务编排 |

`CF_agent-wechat` 是“微信通道入口”，`CF_agent-gateway` 是“进入企业控制面的消息入口”。两者都涉及消息接入，但前者处理渠道适配，后者处理企业级控制和状态，职责不能合并。

## CF_agent-wechat

### 负责

- 托管企业 AI 使用的微信客户端。
- 维护微信登录状态和受控登录管理能力。
- 读取微信私聊与群聊消息，并保留来源会话和消息类型信息。
- 发送文本回复，并提供媒体读取或发送所需的微信适配接口。
- 向 Gateway 暴露受控的消息收发接口。

### 不负责

- 不执行 AI 推理、Agent、模型选择或 Skills。
- 不决定企业身份、权限、Admission、上下文策略或任务路由。
- 不保存 Gateway 的权威任务、响应和投递生命周期状态。
- 不管理正式企业文件、目录、分享、文件权限或持久审计。

它可以接触微信附件字节，但这只属于渠道适配；Attachment 的受控持久化、企业文件归档和文件权限不归该边界负责。

## CF_agent-gateway

### 负责

- 作为企业控制面的消息入口，接收并先持久化来源消息。
- 管理 Checkpoint、消息去重、身份映射、访问策略和 Admission。
- 管理 Workspace、AI Thread、上下文策略、会话绑定和外部 Profile 引用。
- 对获准任务执行路由与分发，并调用当前 AI 执行节点中的 Hermes。
- 管理 Task、Dispatch、Response、Artifact 和 Delivery 等生命周期状态。
- 保存控制面的权威状态，为恢复、日志和审计关联提供依据。
- 把 Hermes 结果先持久化，再编排回原微信会话的投递。

### 不负责

- 不执行 AI 推理、Agent 或 Skills；调用 Hermes 不等于 Gateway 自己生成结果。
- 不创建或管理 Hermes 内部配置档案，只保存和选择外部引用。
- 不承担正式企业文件的目录、分享、权限或存储服务。
- 不得绕过 `CF_filebrowser-enterprise` 直接访问正式文件存储。

Gateway 规划中的私有媒体存储只服务于短期、受控、可恢复的 Attachment/Artifact 中转，不能演变为第二套企业文件系统。该媒体存储当前仍未完成接入。

## Hermes

### 负责

- 作为 AI 执行环境接收 Gateway 已获准、已路由的任务。
- 执行 Agent 与模型调用，并向 Gateway 返回可持久化结果。
- 创建和管理 Hermes 自身的配置档案；Gateway 只持有外部引用。
- 在后续接入完成后，执行获准的 Skills、Windows 侧能力和企业系统调用。
- 在任务所授予的最小权限与确认边界内使用外部能力。

### 不负责

- 不作为员工消息入口，不直接决定来源身份、Admission 或会话策略。
- 不覆盖 CFserver 上的消息、任务、响应、投递或审计权威状态。
- 不绕过 Gateway 直接向微信会话投递结果。
- 不直接挂载、扫描或绕过 File Service 访问正式企业文件。
- 不因文本执行已验证而自动具备媒体、Skills、企业系统或多节点调度能力。

当前只验证 Hermes 的授权文本 Dispatch 与 Response 范围；可靠自启、媒体、Skills 以及 AI Host 完整恢复仍未完成。

## CF_filebrowser-enterprise

### 负责

- 提供唯一正式企业 File Service。
- 提供企业文件和目录管理。
- 执行用户权限、API Token capability、Share capability 和路径安全检查。
- 提供受控分享、WebDAV 与持久审计等文件服务能力。
- 作为 Gateway、Hermes、Skills 和其他自动化客户端未来访问正式企业文件的统一接口。

### 不负责

- 不负责微信登录、消息读取或回复发送。
- 不负责消息上下文、AI Thread、任务路由、Dispatch 或投递状态。
- 不执行 AI 推理、Agent 或业务 Skills。
- 不替代 Gateway 的聊天临时媒体生命周期和消息侧 Attachment/Artifact 状态管理。

“唯一正式 File Service”是已经确定的职责边界，不表示它与 Gateway/Hermes 的端到端接入已经完成，也不表示本仓库已确认其物理部署位置。

## 跨边界调用规则

1. 员工微信消息先经 `CF_agent-wechat` 完成渠道接入，再进入 Gateway 控制面。
2. Gateway 只有在消息持久化、身份映射和权限允许后，才能向 Hermes 分发任务。
3. Hermes 的结果必须回到 Gateway 持久化，再由 Gateway 编排微信投递。
4. 任何正式企业文件操作都必须经过 `CF_filebrowser-enterprise` 的 File Service API、权限检查和持久审计。
5. Skills 和企业系统调用必须继承当前任务的身份、权限、确认、幂等和审计约束。
6. 本仓库只维护系统级边界、状态与路线图，不承载上述组件的业务代码或生产配置。

## 当前接入状态

- 微信通道到 Gateway、Gateway 到 Hermes、Hermes 结果回到原微信会话的授权文本链路已在限定范围内实机验证。
- 媒体 Attachment/Artifact 链路、正式文件端到端接入和业务 Skills 均未完成。
- 当前是单一 AI Host；多 AI 节点调度尚未设计、实现或验证。

详细分类见[V1 当前状态](../status/v1-current-status.md)，固定文件边界见[技术决策记录](../../05_技术决策记录.md)。
