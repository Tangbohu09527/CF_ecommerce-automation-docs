# 员工工作区与 AI 会话线程设计

> 状态日期：2026-08-01。本文定义 Employee Workspace / 员工工作区与 AI Thread / AI 会话线程的设计基线，不代表 Identity Mapping、Employee Conversation Manager、Hermes 员工工作台或端到端链路已经实现。

## 1. 定位与术语

本设计支持“一套或少量统一 Hermes 服务承载多个员工逻辑工作区”。系统不为每个员工部署一套独立 Hermes 进程。每个 Employee Workspace / 员工工作区可以包含多个 AI Thread / AI 会话线程，不同员工的个人对话历史、任务状态和个人上下文默认隔离。

### Physical Conversation / 物理会话

来源平台实际存在的聊天窗口，例如：

- 微信私聊。
- 微信群聊。
- 未来飞书群。
- 未来钉钉会话。

Physical Conversation / 物理会话由 `source.platform + source.account_id + physical_conversation_id` 确定作用域。它负责描述消息从哪里进入、结果应回到哪里，不等于 Employee Workspace / 员工工作区或 AI Thread / AI 会话线程。

### Enterprise Identity / 企业身份

企业内部稳定的员工身份。`enterprise_identity_id` 是 Gateway 内部不可变的企业身份主键，也是身份、Employee Workspace / 员工工作区和权限关联的权威主键。一个 Enterprise Identity / 企业身份未来可以显式绑定一个或多个微信账号、飞书账号、钉钉账号和 Web 账号。

`employee_id` 是可空的公司员工编号、HR 编号或业务人员编号，不是 Gateway 内部主键。来源平台的 `sender.id`、微信 `wxid`、昵称或群名片都不能代替 `employee_id`；平台稳定标识只用于查找权威身份映射，展示名称只用于显示和辅助审计。

### Employee Workspace / 员工工作区

以 Enterprise Identity / 企业身份为所有者的顶层逻辑工作区，聚合该员工的：

- AI Thread / AI 会话线程。
- 历史任务与当前任务。
- 排队、执行、等待、成功和失败等处理状态。
- 个人上下文与上下文快照引用。
- 结果与回传记录。

Employee Workspace / 员工工作区是逻辑容器，不是 Physical Conversation / 物理会话，不是一个 Hermes 进程，也不代表该员工拥有全部 Skill、文件或企业系统权限。

### AI Thread / AI 会话线程

Employee Workspace / 员工工作区内用于隔离一段稳定对话上下文的逻辑线程。一个员工工作区可以有多个 AI Thread / AI 会话线程，例如一个私聊线程和多个“群聊 + 当前员工”线程。

Gateway 生成并保存稳定的 `ai_thread_id`。Physical Conversation / 物理会话负责来源与回传路由，AI Thread / AI 会话线程负责个人上下文和任务顺序；二者不能互相替代。

### Hermes Runtime Thread / Hermes 运行时线程

Hermes 或具体 AI Provider 在执行时使用的线程或 Session 标识，以 `hermes_thread_id` 表示。它不是系统权威主键，可能：

- 尚未创建并保持为 `null`。
- 在首次运行时创建。
- 在 Provider 或模型切换后变化。
- 在 Hermes、Worker Bridge 重启后重新绑定。

Gateway 中稳定的 `ai_thread_id` 才是企业侧权威线程标识。禁止将 `hermes_thread_id` 作为唯一且不可替换的系统主键。

## 2. 权威关系

Gateway / Debian 控制面是以下信息的权威来源：

- Enterprise Identity / 企业身份及来源账号映射。
- Employee Workspace / 员工工作区。
- AI Thread / AI 会话线程及其 Physical Conversation / 物理会话绑定。
- 消息历史与 Context Snapshot / 上下文快照。
- Task 归属、顺序、状态和结果。
- 结果回传的原平台、原账号和原物理会话目标。
- `ai_thread_id` 与 `hermes_thread_id` 的绑定及变更历史。

Hermes、Worker Bridge 和具体 AI Provider 只使用 Gateway 下发的工作区、线程、上下文、权限和任务信息。它们不得：

- 根据昵称、群名片或相似文本自行合并员工。
- 自行决定两个来源账号属于同一员工。
- 自行创建跨员工共享上下文。
- 把 Hermes Runtime Thread / Hermes 运行时线程当成企业权威记录。
- 用 AI 机本地状态覆盖 Gateway 中较新的权威状态。

## 3. 身份映射

Source Identity Mapping / 来源身份映射以 Adapter 和 Message Store 保存的入口事实为输入：

- `source.platform`。
- `source.account_id`。
- `sender.id`。

有效映射输出：

- `enterprise_identity_id`。
- 可选 `employee_id`。

Identity Mapping 不创建或返回 `workspace_id`。`sender.display_name` 可以随消息作为展示快照和辅助审计信息保存，但不是身份映射输入键。

固定原则如下：

- 展示名称不能作为授权、合并或身份主键。
- 微信 `wxid` 等平台稳定标识用于查找映射，但不得直接当作 `employee_id`。
- 映射成功、失败、冲突或已失效的解析结果都必须记录并关联原消息。
- 映射失败时消息及可得附件元数据仍保存在 Message Store，不创建 Task、执行上下文或新的 AI Thread / AI 会话线程执行关系。
- 非白名单用户消息仍保存在 Message Store；身份映射成功不等于 Access Control 允许创建 Task。
- 一个员工可以显式绑定多个来源平台账号。
- 一个来源账号在同一有效期内默认只能映射一个 Enterprise Identity / 企业身份；冲突时拒绝创建 Task。
- 映射新增、修改、停用、冲突处理和有效期变化必须保留审计记录。

Identity Mapping 回答“这个来源账号对应哪个企业员工？”。Access Control 回答“这个员工的这条消息能否创建任务？”。两者是相邻但不同的控制面职责。

Employee Conversation Manager 在两者之后运行：只有身份映射成功且 Access Control 允许创建 Task，才解析或创建 `workspace_id` 和 `ai_thread_id`。

## 4. 工作区模型

`employee_workspace` 是概念实体，不预先限定数据库或物理字段类型。建议字段如下：

| 字段 | 语义 |
| --- | --- |
| `workspace_id` | Gateway 生成的稳定工作区 ID |
| `enterprise_identity_id` | 工作区所有者的不可变权威主键 |
| `employee_id` | 可空的公司员工编号、HR 编号或业务人员编号；不是 Gateway 内部主键，不使用来源平台账号代替 |
| `display_name` | 工作台显示名称快照，不作为授权依据 |
| `status` | `active`、`disabled` 或 `archived` |
| `created_at` | 工作区创建时间 |
| `updated_at` | 工作区最后一次元数据更新时间 |
| `last_active_at` | 该工作区最后一次获准活动时间 |

状态语义：

- `active`：允许在当前权限策略下创建新 Task。
- `disabled`：禁止创建新 Task；既有消息和历史任务仍按审计与保留策略可追踪。
- `archived`：工作区退出日常使用，保留历史恢复与审计所需记录；重新启用必须走受控变更。

工作区状态不授予任何权限。即使工作区为 `active`，Task 仍须经过 Access Control，并只能得到当前 `permission_scope` 与 `allowed_skills` 的交集。

## 5. AI Thread 映射规则

### 私聊

当前已确定的微信私聊线程键保持不变：

```text
bot_account_id + private_chat_id
```

通过 Identity Mapping 和 Access Control 后，私聊消息进入该员工工作区下与此键对应的私聊 AI Thread / AI 会话线程。该线程键不等于员工工作区；一名员工通过其他入口账号发起的会话不会因为昵称相同而自动合并。

### 群聊

当前已确定的微信群聊线程键保持不变：

```text
bot_account_id + group_chat_id + sender_id
```

因此，同一个群里罗明贺与张三分别 `@` 机器人时，默认进入各自 Employee Workspace / 员工工作区中的两个不同 AI Thread / AI 会话线程。整个群不得共用一个 Hermes 个人上下文。

### 多入口扩展

未来扩展到多平台或多机器人账号时，线程绑定键必须同时考虑：

```text
source.platform
+ source.account_id
+ physical_conversation_id
+ enterprise_identity_id（映射完成时）或稳定 sender_id（仅作映射前隔离）
```

微信现有 `bot_account_id` 对应来源账号作用域。增加平台维度是为了避免不同平台、不同机器人账号发生 ID 冲突，不改变现有微信私聊和群聊键的业务含义。

身份映射完成前可以按来源稳定标识保存消息和隔离待处理事实，但不得据此创建 Employee Workspace / 员工工作区 Task。

### 显式协作

跨员工共享线程、代办、共同任务或管理者接管属于后续规划，默认禁止自动合并。未来如支持，必须具有：

- 明确授权。
- 明确参与者。
- 明确共享内容、任务、文件和时间范围。
- 可撤销的访问关系。
- 完整审计记录。

## 6. 私聊与群聊处理流程

私聊目标流程：

```text
私聊消息
  -> Message Store 保存
  -> Identity Mapping
  -> Access Control
  -> Employee Conversation Manager
  -> 解析或创建 Employee Workspace
  -> 解析或创建私聊 AI Thread
  -> Context Builder
  -> Task Queue
  -> Hermes
```

群聊目标流程：

```text
群消息
  -> Message Store 保存
  -> Identity Mapping
  -> Access Control 检查：
       group_allowed
       AND user_allowed
       AND is_mentioned
  -> Employee Conversation Manager
  -> 解析或创建 Employee Workspace
  -> 解析或创建该群 + 该员工对应的 AI Thread
  -> Context Builder
  -> Task Queue
  -> Hermes
```

Identity Mapping 失败、工作区非 `active` 或 Access Control 条件不满足时：

- 消息仍保存。
- 身份解析结果仍记录并关联原消息。
- 不创建 Task。
- 不构建员工 Hermes 上下文。
- 不自动创建新的 Employee Workspace / 员工工作区、AI Thread / AI 会话线程或执行绑定。
- 不进入 Hermes Runtime Thread / Hermes 运行时线程的执行流程。

## 7. 上下文隔离

默认隔离规则：

- 员工 A 的个人私聊历史不得进入员工 B 的上下文。
- 员工 A 在群里的历史任务不得自动进入员工 B 的 AI Thread / AI 会话线程。
- 企业共享知识可在数据、岗位、Skill 和任务权限均允许时供不同员工使用。
- 群聊公共消息只有与当前任务相关，并符合权限和上下文选择规则时，才能作为有限上下文加入。

Context Builder 可以选择的候选上下文包括：

- 当前触发消息。
- 被引用消息。
- 当前任务附件。
- 同一员工在当前 AI Thread / AI 会话线程内的最近必要消息。
- 少量与当前任务有关且允许使用的群聊公共上下文。
- 该员工未完成的关联任务状态。

禁止：

- 无边界发送完整群历史。
- 引入其他员工的私聊历史。
- 因为处于同一群而合并不同员工的 AI Thread / AI 会话线程。
- 将其他员工未授权任务内容、附件或结果带入当前线程。

每次 Task 使用的实际消息、附件、权限与任务状态都固化为 Context Snapshot / 上下文快照。Message Store 中同一条物理消息可以被多个受控快照引用，但每次引用都必须重新满足员工、任务和数据权限边界。

## 8. 工作台界面目标

以下是 AI 机上 Hermes 工作台的目标设计，不表示界面已经实现。未来工作台应支持：

- 按员工显示独立 Employee Workspace / 员工工作区。
- 查看员工名称与 Enterprise Identity / 企业身份。
- 查看私聊和群聊来源的 AI Thread / AI 会话线程。
- 查看当前 Task、`queue_position` 和队列状态。
- 查看执行 Provider 和模型。
- 查看 `started_at` 和 `elapsed_seconds`。
- 查看成功、失败、等待及回传状态。
- 查看任务历史和原微信会话来源。
- 从 Task 跳转到受控 Context Snapshot / 上下文快照。
- 将结果回传到原 Physical Conversation / 物理会话。

界面隔离不等于数据完全复制，也不等于每名员工运行独立 Hermes。Gateway 保持权威状态，AI 机界面只是受控展示与操作界面；界面缓存不得成为唯一状态来源或扩大操作者权限。

## 9. 任务归属

每个 Task 至少关联：

- `enterprise_identity_id`。
- `workspace_id`。
- `ai_thread_id`。
- `source_message_id`。
- `source_conversation_id`。
- `source_account_id`。
- `context_snapshot_id`。
- `task_id`。

可选关联：

- `employee_id`。
- `hermes_thread_id`。
- `selected_provider`。
- `selected_model`。

同一 AI Thread / AI 会话线程内的 Task 默认有序执行，避免两个 Task 并发读取并修改同一上下文。不同员工或不同 AI Thread / AI 会话线程是否并行执行，由 Task Queue、任务风险和 Provider 容量共同决定，不由 Hermes 本地窗口自行调度。

## 10. 结果回传

每个结果必须同时关联：

- Employee Workspace / 员工工作区。
- AI Thread / AI 会话线程。
- Task。
- 原始消息。
- 原始平台账号。
- 原始 Physical Conversation / 物理会话。

结果默认回传原微信私聊或原微信群聊。即使结果已经显示在 Hermes 员工工作区中，也不得丢失 `source.platform`、`source.account_id` 和 `source_conversation_id` 等原始路由信息。

Task 完成与平台发送成功是两个不同状态：

- Task `succeeded` 表示结果已经被 Debian 控制面接收并持久化。
- `result_delivery` 成功表示 Adapter 已把结果发送到原 Physical Conversation / 物理会话。

发送失败不得倒改 Task 的执行事实，也不得伪装为回传成功；系统应保留发送尝试、错误和后续重试或人工处理状态。

## 11. 重启与迁移

Gateway 中的 Employee Workspace / 员工工作区和 AI Thread / AI 会话线程是稳定权威记录。即使发生以下变化：

- Hermes 重启。
- Worker Bridge 重启。
- AI 机更换。
- GPT 模型切换。
- Provider 切换。
- `hermes_thread_id` 变化。

系统仍可根据 Message Store、AI Thread / AI 会话线程、Context Snapshot / 上下文快照和 Task Store 恢复员工工作区、任务历史及原会话回传目标。恢复时可创建新的 Hermes Runtime Thread / Hermes 运行时线程，并追加重绑定记录。

禁止依赖 AI 机本地未同步的聊天窗口、临时缓存或 Provider Session 作为唯一状态来源。

## 12. 数据模型建议

以下均为概念实体，不代表数据库、表结构或迁移已经实现。

### `enterprise_identity`

企业员工身份。`enterprise_identity_id` 是不可变主键；建议同时包含可空 `employee_id`、显示信息、状态、组织目录引用和审计时间。

### `source_identity_mapping`

来源平台账号与 Enterprise Identity / 企业身份的显式映射。建议包含平台、来源账号、稳定发信人 ID、企业身份、有效期、状态、映射依据、变更原因和审计引用。

### `employee_workspace`

员工顶层工作区。建议包含第 4 节所列工作区字段，并以 `enterprise_identity_id` 表达所有者。

### `ai_thread`

员工工作区内的逻辑 AI Thread / AI 会话线程。建议包含 `ai_thread_id`、`workspace_id`、线程类型、状态、创建时间、最后活动时间和当前上下文版本。

### `thread_source_binding`

AI Thread / AI 会话线程与 Physical Conversation / 物理会话的绑定。建议包含 `ai_thread_id`、平台、来源账号、物理会话 ID、稳定发信人或企业身份、线程键版本、有效期和状态。

`hermes_thread_id` 不应取代该实体；运行时绑定应作为可变关联或绑定历史保存。

### `context_snapshot`

某次 Task 实际使用的消息、附件、任务状态、授权快照及选择原因。快照不可变，并关联 `workspace_id`、`ai_thread_id`、`task_id` 和版本。

### `task`

员工、工作区、线程和来源消息归属。除 Task Queue 的状态与调度字段外，至少包含第 9 节列出的归属与来源标识。

## 13. 安全与审计

至少记录并控制：

- 身份映射新增、修改、停用、冲突和有效期变更。
- 工作区禁用、归档和重新启用。
- AI Thread / AI 会话线程归属或来源绑定变更。
- Context Snapshot / 上下文快照选择了哪些消息、附件和状态，以及选择原因。
- 跨员工消息、Task、文件、上下文和结果访问拒绝。
- 管理员查看其他员工工作区的单独授权、理由、范围和时间。
- Task 结果回传目标、发送尝试和最终状态。
- `hermes_thread_id` 创建、失效和重新绑定。

普通日志只记录必要元数据、关联 ID、状态和脱敏错误摘要，不输出完整敏感对话正文、真实附件内容、Token、密码、Cookie 或微信登录数据。审计查询本身也必须受权限控制。

## 14. 当前状态

### 已确定 / 已设计

- Physical Conversation / 物理会话与 AI Thread / AI 会话线程分离。
- 微信私聊线程键为 `bot_account_id + private_chat_id`。
- 微信群聊按 `bot_account_id + group_chat_id + sender_id` 隔离。
- Gateway / Debian 控制面为员工身份、工作区、线程、消息、上下文、任务和回传路由的权威状态中心。
- `enterprise_identity_id` 是身份、工作区和权限关联的不可变权威主键；`employee_id` 是可空业务编号，不是内部主键。
- Identity Mapping 只输出 `enterprise_identity_id` 和可选 `employee_id`；Employee Conversation Manager 在准入允许后解析或创建 `workspace_id` 和 `ai_thread_id`。
- 一个统一 Hermes 服务承载多个 Employee Workspace / 员工工作区，每个工作区可有多个 AI Thread / AI 会话线程。
- `ai_thread_id` 是稳定权威标识，`hermes_thread_id` 只是可空、可替换的运行时绑定。

### 尚未实现

- Identity Mapping。
- Employee Workspace / 员工工作区实体与管理能力。
- Employee Conversation Manager / 员工会话管理器。
- Hermes 独立员工界面。
- `hermes_thread_id` 绑定与重绑定。
- 工作区恢复流程。
- 从微信、Gateway、Task Queue 到 Hermes 和原会话回传的端到端联调。

相关边界见[系统设计](../02_系统设计.md)、[企业 AI Gateway 架构](../architecture/gateway-architecture.md)、[Hermes 事件协议](./hermes-event-schema.md)、[Message Store 设计](./message-store-design.md)、[Task Queue 设计](./task-queue-design.md)和[Access Control 设计](./access-control-design.md)。
