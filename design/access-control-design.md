# Access Control 设计

> 状态日期：2026-08-01。本文定义 Gateway 内部的企业访问控制模块，不是已部署配置或实现说明。当前 Identity Mapping、用户白名单、群权限、Skill 权限、工作区查看权限、策略存储和审计闭环均待实现与验证。

## 1. 模块定位

Access Control 是 CF Gateway 的内部模块，不是独立消息入口，也不是 Gateway 的全部职责。Gateway 先通过 Message Store 保存所有进入系统的消息，由 Identity Mapping 确认来源账号对应的 Enterprise Identity / 企业身份，再调用 Access Control 判断是否创建 AI 任务。

Access Control 只决定：

- 当前消息是否有权创建 Task。
- Task 可以获得哪些 `permission_scope` 和 `allowed_skills`。
- Skill 调用或高风险动作是否满足权限及确认条件。

Access Control 不决定：

- 消息是否保存。白名单、非白名单、未启用群和未 `@` 机器人的消息都必须持久化。
- 来源账号对应哪个企业员工。该事实由 Gateway Identity Mapping 依据权威映射提供。
- Employee Workspace / 员工工作区和 AI Thread / AI 会话线程如何建立或绑定。该职责属于 Employee Conversation Manager。
- 如何选择 AI Provider。Provider 路由由 Gateway AI Router 根据获准任务和路由约束执行。
- 如何理解消息、规划任务或生成回答。这些是 Hermes 的职责。

详细位置见[企业 AI Gateway 架构](../architecture/gateway-architecture.md)，授权快照见[Hermes 事件协议](./hermes-event-schema.md)。

## 2. 设计原则

- **消息先保存：** Message Store 写入成功后再做准入判断；权限拒绝不删除消息历史。
- **拒绝默认：** 没有明确允许规则、关键事实缺失或策略不可用时，不创建 AI 任务。
- **稳定标识：** 权限绑定平台、机器人账号、会话和发信人的稳定 ID，不使用昵称、群名或正文匹配授权。
- **权限前置：** 准入检查早于 Context Builder、Task 创建、Task Queue 和 AI Router。
- **最小权限：** 允许创建 Task 不等于允许所有 Skill、数据、文件或写操作。
- **工作区不提权：** Employee Workspace / 员工工作区只表达归属与隔离，存在、启用或可见都不能扩大 `permission_scope`。
- **拒绝优先：** 显式禁用、过期、身份冲突或高风险条件未满足时，不能被其他允许规则覆盖。
- **可追踪：** 每次允许和拒绝都关联消息、策略版本、决策原因、来源和时间。
- **权威集中：** Debian 保存权限配置和决策记录，Hermes、AI Provider 和 Skill 不维护或修改权限真相。

## 3. 准入判定

### 3.1 私聊

私聊消息创建 AI 任务的前提是 Enterprise Identity / 企业身份映射有效，准入条件为：

```text
sender whitelist = true
```

在事件协议中对应 `authorization.user_allowed=true`。`group_allowed` 和 `is_mentioned` 在私聊中为 `null`，不得用 `true` 填充不适用条件。

### 3.2 群聊

群聊消息创建 AI 任务同样要求 Enterprise Identity / 企业身份映射有效，并同时满足：

```text
group_enabled = true
AND sender_allowed = true
AND is_mentioned = true
```

在事件协议中，`sender_allowed` 对应 `authorization.user_allowed`，`group_enabled` 的判定结果对应 `authorization.group_allowed`。任一条件为 `false` 或无法确认都拒绝创建任务。

当前不定义以下例外：

- 回复过机器人即可免 `@`。
- 群管理员可以绕过用户白名单。
- 同一批次的后续消息自动继承此前 mention。
- 正文包含机器人昵称或 `@` 字符即可视为结构化 mention。

### 3.3 判定结果

| 结果 | Gateway 行为 |
| --- | --- |
| `allowed` | 计算有效 `permission_scope`，构建受控上下文并创建 Task；Task 随后进入 Task Queue |
| `denied` | 消息继续保留在 Message Store，记录权限决策；不构建 Hermes 上下文、不创建 Task、不进入 AI Router |

访问拒绝是权限决策，不是消息格式错误。是否向原会话发送固定拒绝提示仍待运营规则确认，拒绝提示不得调用 Hermes 生成。

## 4. 用户白名单

### 4.1 标识与映射

用户白名单使用以下来源组合定位入口主体：

- `platform`
- `account_id`
- `sender_id`

该组合必须先由 Identity Mapping 显式映射到 `enterprise_identity_id`，白名单才能产生 `user_allowed=true`。企业身份尚未解析时，不得只凭 `sender_name`、手机号片段、群名片或相似昵称自动合并为已授权用户；微信 `wxid` 等平台稳定标识是来源映射键，不是 `employee_id`。

白名单最终授权 Enterprise Identity / 企业身份，而不是孤立的平台昵称。一个员工可以绑定多个来源平台账号，但每个来源账号都必须有独立、可审计的映射；同一来源账号存在多个有效企业身份映射时拒绝创建 Task。

### 4.2 白名单条目

每个概念条目至少包含：

| 字段 | 说明 |
| --- | --- |
| `platform` | 来源平台，如 `wechat`、`feishu`、`dingtalk` |
| `account_id` | 当前机器人或应用账号，避免多账号权限串用 |
| `sender_id` | 平台侧稳定发信人标识 |
| `enterprise_identity_id` | 白名单授权的企业身份；有效启用条目必须完成映射，未解析时为 `null` 且不得放行 |
| `enabled` | 是否启用；只有明确为 `true` 才可得到 `user_allowed=true` |
| `permission_scope` | 该用户可获得的权限范围上限 |
| `valid_from` / `valid_until` | 可选有效期；过期条目拒绝创建任务 |
| `policy_version` | 所属策略版本 |
| `change_reason` | 新增、修改或停用的业务原因 |

新增、停用和修改白名单必须可审计。生产真实用户标识不进入本仓库示例或普通日志。

### 4.3 判定规则

- 条目不存在、被停用、过期、账号不匹配或身份冲突时，`user_allowed=false`。
- `enterprise_identity_id=null` 或映射无法唯一确定时，`user_allowed=false`。
- 同一人员在不同平台或不同机器人账号中的身份必须显式绑定，不自动跨平台放行。
- 白名单不决定消息是否保存，不授予群启用状态，也不替代群聊 mention 条件。
- 白名单不自动授予高风险 Skill、正式文件路径或企业系统写权限。

## 5. 群权限

群权限按以下组合定位：

- `platform`
- `account_id`
- `conversation_id`

群策略至少包含：

| 字段 | 说明 |
| --- | --- |
| `group_enabled` | 当前群是否明确启用 AI；缺失或非 `true` 时 `group_allowed=false` |
| `permission_scope` | 该群允许的权限范围上限，只能保持或收窄用户权限 |
| `allowed_skill_ids` | 可选群级 Skill 允许集合；默认规则必须拒绝优先 |
| `valid_from` / `valid_until` | 可选有效期 |
| `policy_version` | 所属策略版本 |
| `change_reason` | 启用、变更或停用原因 |

群聊规则：

- 群已启用不代表群内所有成员被允许。
- 用户在白名单不代表该用户在任意群或不 `@` 机器人时可以创建任务。
- `is_mentioned=true` 必须来自平台结构化 mention 数据，并确认目标是当前 `account_id`。
- 仅在文本中出现机器人显示名、`@` 字符或相似字符串不能作为 mention 证据。
- 无法确定 mention 时拒绝创建任务，不能默认 `true`。
- 私聊不读取群策略，`group_allowed` 和 `is_mentioned` 均为 `null`。

群文件、引用、回复和连续补充消息是否能关联此前已授权的触发消息，仍需在任务批次设计中单独确定。当前不允许它们隐式继承 mention 或授权决定。

## 6. Skill 权限

### 6.1 权限范围

Skill 权限以稳定 `skill_id` 和动作范围表达，不把自然语言提示词当作授权。概念权限可区分：

- 是否允许调用某个 Skill。
- 只读、创建、修改、删除、批量操作等动作范围。
- 可访问的数据域、业务账号或逻辑文件路径范围。
- 是否需要人工确认，以及确认有效期和绑定参数。

具体 Skill 清单、动作命名和业务权限矩阵仍待逐项设计与验收，本文不声明任何业务 Skill 已经可用。

### 6.2 有效权限计算

有效 `permission_scope` 取所有适用限制的交集：

```text
用户权限上限
INTERSECT 群权限上限（仅群聊）
INTERSECT Gateway 全局策略
INTERSECT 当前任务和风险约束
```

群聊不会扩大用户权限，只能保持或收窄。若交集为空，Task 不得调用任何 Skill；是否允许仅执行不调用 Skill 的受控普通问答，由后续产品策略决定，未明确前不得扩大权限。

Gateway 根据有效范围生成 `allowed_skills` 和 `authorization.permission_scope`。Hermes 可以从 `allowed_skills` 中选择，但不能增加 Skill 或权限。实际 Skill 调用前，Gateway / 权威控制面必须再次检查：

1. 当前 Task、用户和会话仍有效。
2. 所选 `skill_id` 和动作在有效权限范围内。
3. 文件、数据域和业务账号范围匹配。
4. 高风险动作已获得与当前参数绑定的有效人工确认。
5. 幂等键和任务状态允许执行。

高风险确认是附加条件，不会把原本无权的动作变为有权。AI Provider 和 Hermes 都不得跳过第二次检查。

### 6.3 工作区与管理员查看权限

Employee Workspace / 员工工作区存在、状态为 `active` 或出现在 Hermes 工作台中，都不代表拥有全部 Skill、文件、消息或企业系统权限。工作区只能承接 Access Control 已经允许的范围，不能扩大 `permission_scope`、`allowed_skills` 或数据访问范围。

员工默认只能查看和操作自己的工作区及获准内容。管理员查看其他员工工作区必须具有单独权限，并区分工作区列表、任务元数据、Context Snapshot / 上下文快照、消息正文、附件和接管操作等范围；具有系统运维权限不自动获得完整敏感对话查看权。

跨员工查看、拒绝、导出、接管和权限变更必须记录操作者、目标员工、工作区、范围、原因、时间和结果。未来跨员工协作也不得以管理员界面可见为由自动合并 AI Thread / AI 会话线程。

## 7. 权限模型

当前设计采用“显式白名单 + 属性条件 + 明确权限范围”的最小模型，可视为面向 V1 的受限 ABAC，不预设完整 RBAC 已经存在。

| 维度 | 当前属性 | 作用 |
| --- | --- | --- |
| 主体 | 平台、账号、发信人、企业身份、白名单状态 | 确定是谁提出请求 |
| 会话 | 私聊/群聊、群启用状态、mention 状态 | 确定请求从哪里及如何触发 |
| 资源 | AI Task、Skill、企业系统、数据域、文件逻辑路径 | 确定要访问什么 |
| 动作 | 创建任务、调用、读取、创建、修改、删除、批量操作 | 确定要做什么 |
| 条件 | 时间、策略版本、任务状态、人工确认、风险级别 | 确定在什么条件下允许 |
| 决策 | `allowed` / `denied`、原因码、有效范围 | 形成可审计结果 |

### 7.1 决策顺序

1. Adapter 标准化消息，Gateway Message Store 完成持久化。
2. 校验平台、机器人账号、Adapter 身份和消息基础结构。
3. Identity Mapping 使用会话类型和发信人稳定标识解析 Enterprise Identity / 企业身份；无法唯一解析时拒绝默认。
4. 查询有效用户白名单，计算 `user_allowed`。
5. 群聊查询群策略并验证结构化 mention；私聊跳过群条件。
6. 按拒绝默认规则形成 `allowed` 或 `denied` 决策并关联原消息。
7. 对允许消息计算 `permission_scope`，生成带策略版本的不可变授权快照。
8. 仅为允许消息由 Employee Conversation Manager 定位 Employee Workspace / 员工工作区和 AI Thread / AI 会话线程，再构建上下文、创建 Task 并进入 Task Queue。
9. 在 Skill 调用和高风险动作前重新校验有效权限及确认状态。

### 7.2 决策原因

原因码至少需要区分：

| 原因码 | 含义 |
| --- | --- |
| `allowed` | 所有适用准入条件已明确满足 |
| `user_not_allowed` | 用户不存在于有效白名单或条目已失效 |
| `group_not_allowed` | 群未启用 AI、群策略失效或群不匹配 |
| `bot_not_mentioned` | 群消息未明确 `@` 当前机器人 |
| `identity_unresolved` | 无法可靠解析发信人或企业身份 |
| `mention_unresolved` | 平台数据不足，无法可靠判断 mention |
| `policy_unavailable` | 权威策略无法读取或版本无效 |
| `scope_empty` | 有效权限交集为空 |
| `skill_not_allowed` | 所选 Skill 或动作不在有效范围内 |
| `confirmation_required` | 高风险动作缺少有效人工确认 |

用户可见提示不应直接暴露内部原因码、白名单成员、群策略或 Skill 权限清单。

## 8. 配置模型

以下是概念配置对象，不决定数据库、配置中心或文件格式，也不代表配置服务已经实现。

| 配置对象 | 核心键 | 主要内容 |
| --- | --- | --- |
| `gateway_account` | `platform + account_id` | 入口账号状态、所属组织、允许的 Adapter 和全局策略引用 |
| `user_allowlist_entry` | `platform + account_id + sender_id` | 企业身份引用、启用状态、有效期、用户权限上限；有效条目必须能唯一映射企业身份 |
| `group_policy` | `platform + account_id + conversation_id` | `group_enabled`、有效期、群权限上限和群级 Skill 限制 |
| `skill_grant` | 主体或未来角色 + `skill_id` | 允许动作、数据域、文件范围、风险和确认要求 |
| `policy_bundle` | `policy_version` | 一组原子生效的账号、用户、群和 Skill 策略版本 |

配置管理必须满足：

- Debian 是权威来源；Hermes、AI Provider、Skill 或 Adapter 本地缓存不得覆盖更新版本。
- 每次发布使用不可变 `policy_version`，相关配置原子生效，避免用户、群和 Skill 规则只更新一部分。
- 变更记录操作者、时间、原因、前后版本和审批信息；具体审批流程待确认。
- 删除优先采用可追踪停用，避免历史授权决策失去解释依据。
- 策略缓存失效、版本不一致或权威来源不可用时拒绝创建新任务。
- API 密钥、Cookie、微信登录数据和企业系统凭证不属于权限配置，不写入策略正文或仓库。
- 真实用户、群和企业数据不得作为本文档仓库中的配置样例。

## 9. 授权快照与审计

Gateway 在标准事件的 `authorization` 字段记录本次判定快照，至少包含：

- `user_allowed`
- `group_allowed`
- `is_mentioned`
- `permission_scope`
- 最终 `decision`、`reason_code` 和 `policy_version`

该快照描述当时依据哪个策略作出什么决定，不是可由 Hermes 修改的实时配置。策略后续变更不改写历史事件；尚未执行或高风险动作仍须按当前权威策略重新校验。

Message Store 与权限审计是不同记录：

- Message Store 按企业消息历史要求保存授权和未授权消息正文、引用及附件元数据。
- 权限审计保存 `event_id`、主体、会话、决策时间、策略版本、结果、原因码和有效范围摘要。
- 普通运行日志不无条件复制完整消息正文、附件内容或敏感身份信息。

## 10. Hermes 与 AI Provider 边界

- Gateway Access Control 形成授权决定和有效权限范围。
- Employee Workspace / 员工工作区和 AI Thread / AI 会话线程只承载该决定，不得扩大权限。
- AI Router 只能在 Task 的权限与数据约束内选择已批准 Provider。
- Hermes 只消费 Gateway 已允许的 Task、上下文快照、`permission_scope` 和 `allowed_skills`。
- AI Provider 只提供模型或执行能力，不授予用户、群或 Skill 权限。
- Hermes 或 Provider 发现权限缺失时必须停止，但不能把拒绝改为允许。

权限检查发生在 Gateway，不由 Hermes、模型提示词或 Provider 自报能力决定。

## 11. 后续 RBAC 扩展

V1 先使用明确的用户白名单、群策略和 Skill 授权，避免在组织角色和业务权限尚未确认时虚构完整 RBAC。后续可扩展：

- 企业用户与角色的多对多绑定。
- 角色到 Skill、动作、数据域和文件路径的权限集合。
- 部门、岗位、临时项目组和代理授权。
- 角色继承、权限上限、显式拒绝和职责分离。
- 高风险操作的审批人、双人复核和时间条件。
- 基于身份目录的入离职同步、定期复核和权限回收。

RBAC 扩展后，有效权限仍须由 Gateway 计算并写入授权快照。Hermes 只接收最终允许的范围，不解析角色继承，也不自行判断用户应属于哪个角色。

## 12. 当前状态与待验证项

| 项目 | 状态 |
| --- | --- |
| 用户白名单、群权限和 Skill 权限模型 | **本文形成设计基线，待实现和业务确认** |
| Message Store 与权限决策关联 | **目标设计，待实现** |
| 权限配置存储、版本发布和审计 | **待设计实现** |
| 企业身份映射 | **待实现和验证** |
| 工作区权限边界与管理员跨员工查看 | **设计基线，权限项、审批和审计待实现** |
| 微信群结构化 mention 识别 | **待基于 `agent-wechat` 实际样本验证** |
| 飞书、钉钉身份与 mention 映射 | **后续规划，未验证** |
| 完整 RBAC | **后续规划** |

当前只确认了企业级准入规则、工作区权限边界和职责划分，不代表 Identity Mapping、权限系统、Hermes 员工工作台、配置中心、审批流或 RBAC 已经上线。员工归属与线程设计见[员工工作区与 AI 会话线程设计](./employee-workspace-design.md)。
