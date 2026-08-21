# Context Runtime

> [!WARNING]
> **文档状态：2026-08-11 V2 Enterprise Runtime 历史快照。**
> 本文保留该版本当日的架构与实现边界，不代表当前生产状态；正文中的“当前”“已发布”“已启用”等表述均按该版本和原 Staging 环境理解。当前生产事实以[当前状态矩阵](../../status/current-status.md)为准，正式系统架构以[System Architecture](../../architecture/system-architecture.md)为准。

> 适用版本：`v2-enterprise-runtime-20260811`
> 文档状态：与当前已发布版本一致
> 范围：Context Runtime、Timeline、Context Snapshot 与 Memory 的术语和运行边界

## 1. 文档目的

本文定义 `CF_agent-gateway` V2 Enterprise Runtime 中的上下文语义，供开发交接、部署验收和运行排障使用。本文不新增 API、数据表、缓存介质、过期时间或截断算法；具体存储结构和参数以当前发布实现为准。

必须始终区分以下四个概念：

| 概念 | 当前定义 | 是否为事实来源 | 当前状态 |
| --- | --- | --- | --- |
| Context | 当前线程上下文 | 否，由获准事实和当前运行状态构成 | 已发布能力 |
| Timeline | 按时间组织的事实记录 | 是，属于 Gateway / Debian 权威事实边界 | 已发布能力 |
| Snapshot | Context 的可重建缓存 | 否，是派生数据 | 已发布能力 |
| Memory | 未来用于长期认知的独立能力 | 否；当前不存在该运行时 | 后续规划，未实现 |

`Context`、`Timeline`、`Snapshot` 和 `Memory` 不能互换使用，也不能用一个概念的状态推断另一个概念已经存在。

## 2. Context：当前线程上下文

Context 只表示当前 `AI Thread` 执行所需的上下文。它由 Gateway 在当前企业身份、工作区、线程和权限边界内组织，供 Hermes 进行推理和工具选择。

Context 的边界如下：

- 作用域是当前线程，不是全员、全群或全系统消息集合。
- 内容来自当前获准的消息事实、线程关系和必要运行状态，不由 Hermes 自行扩展权限范围。
- Context 可以随当前线程的新事实和运行结果变化，因此它不是不可修改的消息档案。
- Context 不是审计记录；需要追溯时应回到 Message Archive、Timeline 及关联的运行状态。
- Context 不等于 Hermes Provider 的本地 Session。`hermes_thread_id` 可变化，Gateway 管理的线程关系仍是权威关系。

Gateway 负责事实、权限和编排；Hermes 只消费 Gateway 提供的已授权 Context。Hermes 不得根据模型内部历史跨身份、跨工作区或跨线程补充上下文。

## 3. Timeline：事实记录

Timeline 是按时间组织的事实记录，用于说明某个线程或相关运行对象实际发生了什么。它属于 Gateway / Debian 权威状态边界，是上下文恢复、问题追踪和管理查询的依据。

Timeline 与 Context 的区别：

- Timeline 记录事实；Context 选择当前执行需要看到的内容。
- Timeline 的存在不表示其中全部内容都能进入某次 Context，选择仍受身份、线程和权限约束。
- Context 的变化不能覆盖、删改或重新解释 Timeline 中已经持久化的事实。
- Timeline 不是模型提示词，也不是长期认知 Memory。

Message Archive 保存入口消息事实，Timeline 按时间呈现相关事实。两者共同位于权威事实侧；Snapshot 只能从这些事实及获准运行状态派生，不能反向成为事实来源。

## 4. Snapshot：上下文缓存

V2 Enterprise Runtime 中的 Snapshot 是 **Context 的缓存**。它用于减少重复构建当前线程上下文的成本，并支持在缓存有效时恢复相应的 Context 视图。

Snapshot 必须满足以下语义：

- 是派生数据，不是 Message Archive 或 Timeline 的替代品。
- 可以缺失、失效或被重建；其生命周期不改变底层事实。
- 只能在原企业身份、工作区、线程和权限边界内使用。
- Snapshot 与当前权威事实不一致时，以权威事实为准。
- Snapshot 损坏或不可用时，应从 Message Archive、Timeline 和当前获准状态重建，不能把 Hermes 本地会话当作恢复来源。
- Snapshot 不是长期 Memory，也不产生自动记忆能力。

### 4.1 与早期设计术语的关系

仓库早期设计曾以概念实体 `context_snapshot` 描述某次执行使用的不可变输入记录。本文针对当前发布的 V2 Runtime，`Context Snapshot` 明确定义为 **上下文缓存**。维护者不得仅凭早期概念实体名称，推断 V2 Snapshot 具有不同的持久化或不可变语义；实际数据映射和兼容行为以 `v2-enterprise-runtime-20260811` 的发布实现为准。

无论术语历史如何，当前边界不变：Snapshot 不替代 Timeline 事实，也不是 Memory。

## 5. Memory：未来长期认知

Memory 是未来可能用于跨时间、跨会话保留长期认知的独立能力。它需要单独的数据来源、写入规则、权限、纠错、删除、保留和审计设计，不能由消息归档、Timeline 或 Snapshot 自动推导为已经存在。

当前版本 **未实现 Memory Runtime**，因此：

- 不承诺自动提取、沉淀或更新用户偏好和长期知识。
- 不承诺跨线程自动回忆。
- 不把 Snapshot 保留时间长解释为 Memory。
- 不把 Hermes Provider 的会话历史解释为企业长期 Memory。
- 不允许以“记忆”为理由绕过当前线程和数据权限边界。

## 6. 运行关系

以下流程只说明职责关系，不定义数据库字段或具体缓存算法：

```text
Message Archive / Timeline（权威事实）
              ↓
      权限与线程边界检查
              ↓
  Context Runtime 构建当前线程 Context
              ↕
   Context Snapshot（可重建缓存）
              ↓
        Hermes 推理与工具选择
              ↓
   Gateway 持久化响应和运行事实
```

Context Runtime 可以在执行准备和响应处理过程中读取或更新 Context 与 Snapshot，但不会因此取得 Gateway 之外的事实权威。响应只有经过 Gateway 的 Response Persistence 后，才成为后续投递与追踪所依据的持久化状态。

## 7. 隔离与权限

- `enterprise_identity_id` 是企业身份、工作区和权限关联的权威主键。
- Context 必须绑定当前获准的工作区和 AI Thread，不得按昵称、群名或相似内容合并身份。
- 同群其他员工的个人消息、任务、附件或结果不得自动进入当前 Context。
- 管理查询可读取的归档范围不等于 Hermes 可使用的上下文范围。
- 管理员查看其他员工的敏感内容需要相应的单独查看权限；一般系统运维权限不自动获得完整对话读取权。
- 正式文件仍须经过 File Service 的权限检查和审计，Snapshot 中的文件引用不能放大文件访问权限。

## 8. 缓存异常与恢复

| 情况 | 处理原则 |
| --- | --- |
| Snapshot 缺失 | 在当前身份、线程和权限范围内依据权威事实重建 |
| Snapshot 失效 | 丢弃或更新派生缓存，不改写 Timeline |
| Snapshot 与事实不一致 | 以 Message Archive、Timeline 和 Gateway 当前状态为准 |
| 无法读取权威事实 | 不把旧 Snapshot 或 Hermes 本地状态提升为权威事实；按当前实现进入失败或排障流程 |
| 线程归属无法确认 | 停止构建可执行 Context，不猜测或跨线程拼接 |

具体日志、健康检查和恢复步骤见 [Runtime 运维](../operations/runtime-operations.md)。

## 9. 当前未实现能力

`v2-enterprise-runtime-20260811` 当前没有实现以下能力：

- RAG
- embedding
- vector database
- automatic memory
- Memory Runtime

因此，当前 Context Runtime 文档不得被用于宣称已经具备企业知识库检索、语义向量召回、自动长期记忆或 Memory 治理能力。相关能力如进入实施，必须先形成明确技术决定，并补齐数据来源、权限、删除、审计和运维边界。

## 10. 维护检查点

变更或排障时至少确认：

- 当前操作针对的是 Context、Timeline、Snapshot 还是未来 Memory。
- Context 是否严格限制在正确的 `enterprise_identity_id`、工作区和 AI Thread。
- Snapshot 是否仍被当作可重建缓存，而非事实来源。
- Snapshot 失效后是否能依据权威事实恢复。
- Hermes 是否只接收 Gateway 已授权的 Context。
- 文档和界面是否没有把 RAG、embedding、vector database 或 automatic memory 标记为已实现。

相关架构见 [V2 Enterprise Runtime 架构总览](../architecture/v2-enterprise-runtime.md)，消息事实边界见 [Message Store 设计](../../design/message-store-design.md)，线程隔离见 [员工工作区与 AI 会话线程设计](../../design/employee-workspace-design.md)，当前限制见 [当前限制](../status/current-limitations.md)。
