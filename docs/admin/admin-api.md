# Admin Archive API

> [!WARNING]
> **文档状态：2026-08-11 V2 Enterprise Runtime 历史快照。**
> 本文保留该版本当日的架构与实现边界，不代表当前生产状态；正文中的“当前”“已发布”“已启用”等表述均按该版本和原 Staging 环境理解。当前生产事实以[当前状态矩阵](../../status/current-status.md)为准，正式系统架构以[System Architecture](../../architecture/system-architecture.md)为准。

> 2026-09-03 当前分类：下文“只读、不提供写操作”只适用于 2026-08-11 快照。Gateway main 当前还实现了受认证的 `uncertain` Dispatch inspection、`retry-approved`、`mark-dead` 和 evidence-backed `confirm-success`，并通过自动化测试/CI；完整生产动作覆盖仍需分别留证。见[Gateway 架构](../../architecture/gateway-architecture.md)和[Production Closeout](../../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。

> 适用版本：`v2-enterprise-runtime-20260811`
> 文档状态：与当前已发布版本一致
> 范围：Admin Archive API 的只读查询能力、权限边界和使用约束

## 1. 定位

Admin Archive API 是 `CF_agent-gateway` V2 Enterprise Runtime 的只读管理查询面，用于在获准范围内查看归档与运行事实。它不参与消息接入、路由、线程创建、Hermes 执行或响应投递，也不提供运行时写操作。

本文只描述能力类别和安全边界，不定义或猜测 URL、HTTP 方法、参数名、字段名、状态码或响应结构。实际接口契约以 `v2-enterprise-runtime-20260811` 的发布实现及其 OpenAPI 为准。

## 2. 权限边界

- 调用主体必须通过当前部署的身份认证，并具有 `admin role`。
- `admin role` 是进入管理查询面的必要条件，不应被解释为绕过数据权限、审计或脱敏规则的通用凭证。
- 管理员跨员工查看工作区、消息正文或其他敏感内容时，必须满足对应的单独查看权限；一般系统运维权限不自动获得完整敏感对话读取权。
- 查询必须受当前组织、企业身份、会话和数据范围约束，不得通过展示名称或模糊匹配扩大访问范围。
- API 凭证、Cookie、访问令牌、微信登录数据及其他秘密不得写入查询示例、普通日志或工单正文。
- 生产部署应对管理查询入口实施最小暴露、访问控制和符合当前实现的日志或审计策略。

非 `admin role` 主体不得通过该 API 读取归档数据。具体认证方式、权限声明和拒绝响应以发布实现及 OpenAPI 为准。

## 3. 只读查询能力

当前 Admin Archive API 只说明以下四类查询能力：

| 查询类别 | 用途 | 只读边界 |
| --- | --- | --- |
| `messages` | 查看 Message Archive 中的消息事实 | 不修改正文、来源事实、授权结果或归档状态 |
| `timeline` | 按时间查看相关事实记录 | 不追加、删除、重排或重写事实 |
| `threads` | 查看 Gateway 管理的线程及其关联状态 | 不创建、合并、迁移、解绑或重置线程 |
| `delivery` | 查看已持久化响应的投递相关状态 | 不触发重投、不修改投递结果、不重新调用 Hermes |

各查询实际返回的内容、可见字段和关联深度以发布实现为准。以上类别名称不能直接推导为 URL 路径、资源名称或数据库表名。

### 3.1 messages

用于定位和核对 Gateway 已归档的消息事实。查询消息不等于把消息送入 Hermes，也不改变该消息的路由、授权或处理状态。

Message Archive 的来源事实、权限决策和任务执行属于不同职责域。管理查询可以用于关联排障，但不得用查询结果覆盖权威记录。

### 3.2 timeline

用于按时间查看与查询范围相关的事实记录。Timeline 是事实视图，不是当前线程 Context、Context Snapshot 或长期 Memory。

Timeline 查询不能追加或修正事实。发现数据异常时，应通过既有运行或维护流程处理，不能将 Admin Archive API 当作数据修复入口。

### 3.3 threads

用于核对企业身份、来源会话与 Gateway 内部线程的关联。`enterprise_identity_id` 是内部企业身份关联的权威主键；展示名、昵称和群名不作为授权或线程合并依据。

线程查询只提供观察能力。它不创建新的 AI Thread，不改变 `hermes_thread_id` 绑定，也不提供跨员工线程合并能力。

### 3.4 delivery

用于查看响应投递相关记录和当前可查询状态，以区分响应已持久化、投递处理中、投递成功或投递异常等运行阶段。实际状态值以发布实现为准，本文列举的阶段说明不构成枚举契约。

Delivery 查询不会重新发送响应。投递失败应按 [Runtime 运维](../operations/runtime-operations.md) 执行排障，并以已持久化响应为基础处理，不能仅因渠道失败重新触发 Hermes 推理。

## 4. 查询过滤

当前查询范围支持以下过滤维度；具体组合方式、参数名、格式、默认值和限制以发布实现及 OpenAPI 为准。

### 4.1 time

- 用于限定需要检查的时间范围，避免无边界读取归档。
- 时间格式、时区、边界是否包含以及默认排序均不得由本文推断。
- 运维查询应尽量使用能覆盖故障窗口的最小时间范围。

### 4.2 identity

- 用于按企业身份缩小查询范围。
- 身份关联以 `enterprise_identity_id` 为权威主键；可选业务编号或来源账号的支持情况以实际契约为准。
- 不得使用昵称、群名片或相似名称代替稳定身份并据此扩大数据访问。

### 4.3 conversation

- 用于按 Physical Conversation / 物理会话缩小查询范围。
- 会话标识必须在相应平台和来源账号作用域内解释，不能假设跨平台全局唯一。
- 同一物理群聊可能对应不同员工的独立 AI Thread；按 conversation 查询不自动授予跨员工敏感内容查看权。

### 4.4 pagination

- 用于分批读取查询结果，避免一次返回无边界数据。
- Cursor、页码、页大小、排序稳定性和最大限制以 OpenAPI 及当前实现为准。
- 调用方不得自行拼接、修改或猜测分页状态；应完整使用服务端返回的分页信息。
- 翻页过程中仍须保持相同的身份和权限上下文。

## 5. 契约来源与调用原则

维护者在调用 Admin Archive API 前应：

1. 确认目标环境运行的版本为 `v2-enterprise-runtime-20260811`，或明确记录版本差异。
2. 从该部署对应的发布实现取得 OpenAPI。
3. 以 OpenAPI 确认实际路径、认证要求、请求参数、过滤格式、分页方式和响应结构。
4. 使用具备 `admin role` 且范围最小的管理身份。
5. 先使用 time、identity 或 conversation 缩小范围，再按服务端分页信息查询。
6. 将查询用于观察和诊断，不把只读 API 当作修复、重放或控制入口。

文档、脚本和监控不得根据本节的概念名称硬编码未在 OpenAPI 中确认的 endpoint 或字段。

## 6. 明确不提供的能力

Admin Archive API 不提供：

- 创建、修改或删除 messages、timeline、threads 或 delivery 记录。
- 修改身份映射、角色、权限、路由规则或授权决定。
- 创建、合并、迁移、重置或接管线程。
- 重放消息、重新分发任务或重新调用 Hermes。
- 触发响应重投、伪造投递成功或覆盖失败原因。
- 导出全部敏感对话的隐含权限。
- RAG、知识库检索、向量检索或 Memory 查询能力。

若后续需要写操作或独立管理能力，必须以新的授权、审计和 API 契约单独设计，不能扩张当前只读接口的含义。

## 7. 运维核对

使用 Admin Archive API 排障时至少确认：

- 查询主体确实具有当前环境认可的 `admin role` 和所需数据查看范围。
- 使用的是目标部署对应的 OpenAPI，而不是其他版本的示例。
- time、identity、conversation 和 pagination 过滤没有意外扩大查询范围。
- `messages` 与 `timeline` 被当作事实查询，Context Snapshot 没有被当作权威记录。
- `threads` 查询没有被误用为线程变更操作。
- `delivery` 查询没有被误用为重投或重新推理操作。
- 查询结果和日志中没有泄露密钥、Cookie、令牌或不必要的敏感正文。

整体职责见 [V2 Enterprise Runtime 架构总览](../architecture/v2-enterprise-runtime.md)，Context 术语见 [Context Runtime](../context/context-runtime.md)，身份与管理员查看边界见 [Access Control 设计](../../design/access-control-design.md)。
