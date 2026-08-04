# Message Store 设计

> 状态日期：2026-08-04。本文定义 CF Gateway 的 Message Store 设计基线。V1 Staging 已通过真实微信文本验证 Polling / Checkpoint、Message Store、Identity / Permission Admission、Employee Workspace / AI Thread、Hermes API 和原会话回复。附件正式处理与完整存储策略、Context Builder、Task Queue、完整 Worker Bridge、Skill、生产数据库和生产部署仍待实现或验证。详见[Gateway V1 Staging 验证记录](../status/gateway-wechat-staging-validation.md)。

## 1. 定位

Message Store 是企业 AI 的长期 Physical Conversation / 物理会话消息历史中心，也是上下文检索、权限追踪和审计的基础。它位于消息标准化之后、Identity Mapping 和 Access Control 之前，权威记录保存在 Debian。

固定处理顺序为：

```text
消息进入 Gateway
  -> 标准化并持久化
  -> Identity Mapping
  -> Access Control
  -> 决定是否创建 Task
  -> 只有获准 Task 才进入 Hermes
```

除 Polling 已确认并在 sink 前过滤的 `is_self=true` 机器人自发消息外，所有成功进入 Gateway 入站处理的消息都必须保存，包括白名单与非白名单用户消息、私聊与群聊消息、群内未 `@` 当前机器人的消息，以及最终不会进入 Hermes 的消息。权限只决定后续是否创建 Task，不决定是否保留聊天记录。self 过滤不是权限拒绝；它推进 Checkpoint，但不进入 Message Store 或执行链。

Message Store 与 Task Store 是两个独立的权威数据域：Message Store 保存全部消息；Task Queue / Task Store 只保存通过准入检查后创建的 AI 任务。一条消息可以没有对应 Task，一条 Task 也可以引用同一获准批次中的多条消息。详细任务边界见[Task Queue 设计](./task-queue-design.md)。

Employee Workspace / 员工工作区和 AI Thread / AI 会话线程映射使用独立实体，不通过复制每名员工的整份消息历史实现隔离。Message Store 保存来源事实，工作区、线程、Context Snapshot / 上下文快照和权限记录决定某次 Task 可以引用哪些消息。

V1 Staging 已将真实微信文本从 Polling / Checkpoint 串入 Message Store、Identity / Permission Admission、Employee Workspace / AI Thread、Hermes API 和文本结果回传。未配置身份的消息不得进入 Hermes；获准测试身份可建立 Hermes Runtime Thread 绑定并收到原会话回复。附件正式处理、Context Builder、Task Queue、完整 Worker Bridge 和 Skill 仍未完成，也不代表生产部署。

## 2. 消息模型

### 2.1 `messages` 核心实体

以下是概念字段，不预先限定关系型数据库或字段物理类型：

| 字段 | 语义 |
| --- | --- |
| `id` | Gateway 生成的稳定内部消息 ID；是 Message Store 内部关联使用的主标识 |
| `event_id` | 首次形成该消息记录的标准事件 ID；同一逻辑事件重投时保持不变 |
| `source` | 来源平台、接入 Provider 和机器人 / 应用账号的结构化标识 |
| `source_message_id` | 来源侧消息 ID；在来源平台、来源账号和会话范围内解释，不作为 Gateway 全局 ID |
| `conversation_id` | 归一化后的物理会话 ID |
| `conversation_type` | `private` 或 `group`；无法确认时隔离处理，不猜测 |
| `conversation_name` | 来源当时提供的会话展示名快照；可为空，不作为权限依据 |
| `sender_id` | 来源账号范围内稳定的发送人 ID |
| `sender_name` | 来源当时提供的发送人展示名快照；可为空，不作为授权依据 |
| `is_mentioned` | 标准化 mention 来源事实；微信仅当原始 `isMentioned` 严格为 `true` 时为 `true`，字段缺失为 `false` |
| `is_self` | 标准化的消息是否由当前来源账号自身发送的事实；当前微信 Polling 在 sink 前过滤 `true`，因此这类消息通常不进入 Message Store |
| `message_type` | 统一消息类型，如 `text`、`file`、`image`、`voice`、`reply`、`forward` |
| `content` | 标准化正文、外层标题或结构化内容引用；不得包含附件二进制 |
| `timestamp` | 来源消息时间；不可用时明确为空，不用 Gateway 接收时间冒充 |
| `reply_to_message_id` | 已解析时指向被引用消息的 `messages.id`；未解析时为空 |
| `authorization_result` | Gateway 在持久化后追加的准入结果，如 `not_checked`、`allowed` 或 `denied`，并关联独立权限审计记录 |
| `created_at` | Gateway 首次持久化记录的时间 |
| `updated_at` | Gateway 最后更新标准化补充字段、附件状态或授权关联的时间 |

建议同时保存 `source_timestamp_raw`、`received_at`、`raw_payload_ref`、`normalization_version`、`identity_resolution_ref`、`authorization_decision_ref` 和 `trace_id`，用于解释来源时间、身份解析、映射版本和处理链路。受控原始载荷与标准消息分开保存，通过不透明引用关联；普通日志不得代替 Message Store。这些建议字段不表示当前代码已经实现。

### 2.2 标识与幂等

`messages.id`、`source_message_id` 和 `event_id` 必须分离：

- `id` 是 Gateway 内部 ID，不暴露来源平台的作用域限制。
- `source_message_id` 是来源平台事实，可能只在某个机器人账号或会话内唯一。
- `event_id` 标识不可变事件；同一消息的附件获取、权限决策或状态变化可以产生新的事件，但不创建新的消息记录。

当前实现已以来源账号隔离来源物理消息唯一性，并以 `event_id` 建立事件幂等约束；两层幂等同时生效。概念来源键为 `source.platform + source.account_id + conversation_id + source_message_id`，相同来源消息标识不得跨来源账号合并。若未来来源不提供稳定消息 ID，Adapter 必须记录该事实，并使用经验证的来源字段和内容指纹形成受控幂等键；该退化键仍是待实现设计，只用于防重，不能伪装成平台原生 ID。

重复投递采用幂等更新：返回已有 `messages.id`，补齐此前未知但本次已确认的字段，不用空值覆盖已有事实，也不重复创建附件、权限决策或 Task。消息去重、附件下载去重、Task 幂等和 Skill 业务写入幂等分别处理。

### 2.3 私聊、群聊与多入口

- `source` 必须包含平台和来源账号，使相同会话或发送人 ID 不会跨账号、跨平台串联；当前代码已实现来源账号隔离，真实微信文本消息接线已通过 Debian Staging 验证，附件接线仍未完成。
- 私聊和群聊共用 `messages` 模型，但权限条件、AI 线程映射和上下文选择规则不同。
- `conversation_name`、`sender_name` 只保存接收时的展示快照；后续改名不改写历史身份事实。
- 微信、飞书、钉钉及未来 API 入口由各自 Adapter 解释原始字段，再映射到统一模型；平台私有字段保存在受控原始载荷中，不扩散为核心字段。
- 无法可靠确认会话类型、发送人或消息类型时，仍保留可得的原始载荷引用和错误记录，但拒绝创建 Task。

### 2.4 员工工作区与消息引用边界

Message Store、Employee Workspace / 员工工作区和 AI Thread / AI 会话线程保持不同的数据职责：

`enterprise_identity_id` 是 Gateway 内部不可变的企业身份主键，也是身份、工作区和权限关联的权威主键。`employee_id` 是可空的公司员工编号、HR 编号或业务人员编号，不是 Gateway 内部主键，也不得使用微信 `wxid` 代替。

- `messages` 记录 Physical Conversation / 物理会话中的来源事实。
- `source_identity_mapping` 记录 `source.platform + source.account_id + sender.id` 到 `enterprise_identity_id` 及可选 `employee_id` 的显式映射；不创建或返回 `workspace_id`。
- `identity_resolution` 记录每条消息的解析状态、`enterprise_identity_id`、可选 `employee_id`、映射版本和原因。
- `employee_workspace`、`ai_thread` 和 `thread_source_binding` 记录员工归属、逻辑线程与来源绑定。
- `context_snapshot` 记录某个 Task 实际选择的消息、附件、权限和任务状态引用。

同一条物理消息可以被多个受控 Context Snapshot / 上下文快照引用，例如当前群消息同时是两项获准任务的必要公共背景；系统不得因此复制或改写原始消息。每次引用必须记录 `enterprise_identity_id`、`workspace_id`、`ai_thread_id`、Task、选择原因和当时权限，且不得跨员工使用未授权的私聊、任务或附件。

Identity Mapping 失败或 Access Control 拒绝只阻止创建员工 Task，不删除 Message Store 记录或身份解析结果，也不创建执行上下文或自动建立新的 AI Thread / AI 会话线程执行关系。只有身份映射成功且 Access Control 允许创建 Task 后，Employee Conversation Manager 才解析或创建 `workspace_id` 和 `ai_thread_id`。完整归属与隔离规则见[员工工作区与 AI 会话线程设计](./employee-workspace-design.md)。

当前“持久化 -> 身份解析 -> Access Control -> Admission -> 工作区 / AI Thread -> Hermes API -> 文本回传”已通过 Debian Staging 真实微信消息验证，`hermes_thread_id` 运行时绑定已建立。Context Snapshot、Task Queue、完整 Worker Bridge、附件和 Skill 运行链路仍未完成。Gateway V1 群聊 whole-room thread 与既定 `group + sender` 隔离设计的偏差，以[Gateway 验证记录](../status/gateway-wechat-staging-validation.md)为准。

## 3. 附件模型

### 3.1 `attachments` 核心实体

| 字段 | 语义 |
| --- | --- |
| `id` | Gateway 生成的稳定附件 ID |
| `message_id` | 所属 `messages.id`；附件必须先关联消息，不要求消息存在 Task |
| `filename` | 来源文件名快照；不作为唯一键或直接存储路径 |
| `file_type` | 统一业务类型，如 `image`、`document`、`archive` 或 `unknown` |
| `mime_type` | 经过来源信息或内容检测确认的 MIME 类型；未知时为空 |
| `file_size` | 已确认的字节数；获取前未知时为空 |
| `storage_path` | Debian 受控存储中的逻辑路径或不透明存储键；未获取、已清理或不允许落盘时为空，不得使用任意主机路径 |
| `hash` | 文件成功获取后计算的带算法内容哈希；未获取时为空 |
| `source_media_id` | 来源平台提供的媒体 / 文件标识；不可用时为空 |
| `fetch_status` | 附件获取状态，如 `pending`、`available`、`failed`、`not_fetched` 或 `purged` |
| `created_at` | 附件元数据首次登记时间 |

建议增加 `updated_at`、`source_attachment_id`、`failure_code`、`failure_message`、`retention_class`、`purged_at` 和 `storage_ref`。其中 `storage_ref` 可作为对外事件使用的不透明引用；`storage_path` 只在受控存储边界内解释，Hermes 和 Skill 不得据此遍历主机文件系统。

### 3.2 元数据与二进制边界

- 附件二进制不得直接写入 `messages.content`、`messages` 表、标准事件 JSON 或普通日志。
- 消息记录和附件元数据必须保留，即使附件获取失败、发送人未获准创建 Task，或附件最终不会进入 Hermes。
- 附件记录是否可进入 Hermes 由准入决定、上下文选择和任务权限共同决定，与附件元数据是否登记解耦。
- 附件二进制是否获取、长期落盘、归档或清理由独立的存储、容量、安全扫描、数据分级和保留策略决定；相关决定必须保留状态与审计记录。
- `authorization_result=denied` 只表示不创建 AI 任务，不能被实现为删除消息、删除附件元数据或自动清空历史记录。
- 二进制因容量、安全或保留周期被清理时，附件记录继续存在，`fetch_status`、`purged_at` 和原因可追踪；Task 不得继续使用失效引用。

## 4. 引用关系

引用消息同时保留结构关系和来源当时可见的摘要：

- 能解析到已存消息时，`reply_to_message_id` 指向被引用消息的 `messages.id`。
- 不能解析稳定 ID 时，`reply_to_message_id=null`，另存来源引用 ID、引用类型、原始引用摘要和 `resolution_status=unresolved`。
- 引用文本保存来源实际提供的脱敏摘要，不通过相似文本猜测原消息。
- 引用文件通过附件记录或附件引用关联，不能把文件二进制嵌入引用正文。
- 当前消息正文、当前附件与被引用内容分别保存，不能因消息类型是 `reply` 而丢失当前消息。

后续解析出原消息时可追加解析结果和审计事件，但不得改写来源当时提供的原始引用摘要。

## 5. 合并转发边界

当前合并转发只按已验证能力保存外层事实：

- 外层 `forward` 消息进入 `messages`。
- 保存外层发送人、标题、来源类型和实际可得的附件元数据。
- 记录 `is_expanded=false`、解析版本和未解析原因。
- 不生成未经来源证实的内部消息、内部发送人或内部文件记录。

内部聊天记录当前未解析。后续可由独立 `forward parser` 增强，将解析出的内部记录保存为带父子关系和解析来源的结构化实体；Parser 失败不能删除或覆盖已经保存的外层消息。

## 6. 查询与审计

Message Store 需要支持未来以下访问路径：

- 按来源账号、会话和时间顺序查询消息历史。
- 按稳定发送人 ID、企业身份映射和时间范围查询。
- 按 `event_id`、`trace_id`、来源消息 ID、Gateway 消息 ID 或附件哈希定位链路。
- 为 Context Builder 检索最近有限条且与当前任务相关的获准消息。
- 追踪权限决策、Task、上下文快照、附件、结果和回传记录。
- 在权限允许的数据范围内支持搜索、审计和知识沉淀。

建议索引至少覆盖来源幂等键、`conversation_id + timestamp`、`sender_id + timestamp`、`event_id`、`authorization_result` 和附件 `message_id` / `hash`。搜索索引和知识库不是权威消息副本；索引可重建，删除、归档和权限变更必须同步处理。

查询 Message Store 不等于向 Hermes 提供全部历史。Context Builder 只在当前 `workspace_id`、`ai_thread_id` 和权限边界内选择当前触发消息、被引用消息、与任务相关的附件，以及最近有限条且相关并获准的历史消息，并生成不可变 `context_snapshot_ref`。完整群聊历史不得无边界发送给 Hermes，其他员工的个人历史和未授权 Task 内容不得进入当前快照。

## 7. 保留与安全

- 按消息正文、附件元数据、附件二进制、原始载荷、搜索索引和审计记录分别制定保留策略；具体周期、归档层级和业务 / 合规要求待确认。
- 保留策略按来源、会话、数据分类和法定保留要求执行，不因是否创建 Task 而隐式缩短消息记录寿命。
- 清理采用可审计流程，记录对象、规则版本、执行时间、结果和操作者；法律保全或调查冻结优先于常规清理。
- 敏感正文、身份、附件和原始载荷采用最小权限访问、传输加密、静态加密和查询审计；搜索结果也必须执行相同权限边界。
- API Token、密码、Cookie、微信登录数据和其他秘密不得写入消息内容副本、普通日志、错误详情或文档示例；检测到疑似秘密时按安全策略隔离和告警，不在日志中回显原值。
- 附件设置单文件大小、会话 / 用户配额、总容量水位、危险类型、压缩包展开和恶意内容限制；超过限制时保留消息和附件元数据并记录安全状态。
- 在线消息库、归档库、附件存储和备份的删除边界必须分别定义。归档不是删除，删除数据库行也不能替代附件对象、索引和备份的生命周期处理。

相关权限规则见[Access Control 设计](./access-control-design.md)，员工归属见[员工工作区与 AI 会话线程设计](./employee-workspace-design.md)，Gateway 总体位置见[企业 AI Gateway 架构](../architecture/gateway-architecture.md)，标准事件字段见[Hermes 事件协议](./hermes-event-schema.md)。
