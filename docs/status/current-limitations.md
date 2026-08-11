# CF_agent-gateway V2 当前限制

> 适用版本：`v2-enterprise-runtime-20260811`
> 状态日期：2026-08-11
> 文档状态：当前已知限制，不代表未来能力已经实现

## 1. 文档目的

本文记录 `CF_agent-gateway` V2 Enterprise Runtime 当前已知的实现和验证边界，供交接、部署验收与运行维护使用。限制项描述的是当前版本不能承诺的能力；“后续方向”仅表示规划或优化方向，不是当前发布能力。

## 2. 限制总览

| # | 当前限制 | 当前状态 | 主要影响 |
| --- | --- | --- | --- |
| 1 | Windows symlink 测试 skip | 验证缺口 | Windows 测试通过不能证明 symlink 场景已验证 |
| 2 | Hermes remote transport 未独立抽象 | 实现边界 | 当前 Hermes Integration 不能表述为通用 remote transport |
| 3 | Skill Runtime 未实现 | 未实现 | 当前版本不能承诺受控企业 Skill 执行运行时 |
| 4 | Memory Runtime 未实现 | 未实现 | 当前版本没有长期认知或 automatic memory |
| 5 | WDT / S6 未接入 | 未集成 | 当前版本不能执行或验证 WDT / S6 业务闭环 |
| 6 | Dispatch 与 Response 事务桥后续优化 | 后续优化 | 当前不得宣称分发到响应全阶段构成单一原子事务 |

## 3. 详细说明

### 3.1 Windows symlink 测试 skip

**现状**

Windows 环境中的 symlink 相关测试当前被 skip。现有 Windows 测试结果只能说明实际执行的测试项通过，不能覆盖被跳过的 symlink 行为。

**运维影响**

- 不得把 Windows 测试通过等同于 symlink 场景已经跨平台验收。
- 涉及 symlink、安全路径边界或文件挂载的部署变更，需要在目标 Debian 环境单独核对。
- 排障记录应保留 skip 项，不能从测试摘要中隐去验证缺口。

**当前处置**

- 保持生产正式文件访问必须经过 File Service、权限检查和审计的边界。
- 在 Debian Staging 或等价 Linux 环境执行发布版本支持的相关验证，并记录环境和结果。
- 未完成目标环境验证前，不扩大 symlink 兼容性声明。

**后续方向**

补齐适用于目标环境的 symlink 测试与发布验收。该方向不表示当前 Windows 测试已经覆盖相关行为。

### 3.2 Hermes remote transport 未独立抽象

**现状**

当前版本已经提供 Hermes Integration，但 remote transport 尚未作为独立、通用的传输层抽象。现有集成契约只应按当前发布实现理解。

**运维影响**

- 不能假设可无改动替换远程传输协议、Hermes 端点形态或其他 Provider。
- 连接、超时和错误行为应依据当前实现和部署配置诊断，不套用不存在的通用 transport 契约。
- 架构文档不得将当前集成宣传为已经完成的可插拔 remote transport。

**当前处置**

- 使用 `v2-enterprise-runtime-20260811` 已发布的 Hermes Integration 路径。
- 变更连接配置前核对当前实现、健康状态和日志，并执行相应回归验证。
- Gateway 继续负责事实、权限和编排；Hermes 只负责推理和工具选择。

**后续方向**

是否拆分 remote transport、如何定义重试和错误契约，需在后续设计与实现中明确；当前不预设接口形态。

### 3.3 Skill Runtime 未实现

**现状**

当前版本未实现独立的 Skill Runtime。Hermes 负责推理和工具选择，不等于企业 Skill 的受控执行、生命周期、权限复核和审计运行时已经具备。

**运维影响**

- 不能将当前 V2 Runtime 用作已完成的企业业务 Skill 执行平台。
- 不得宣称文档处理、浏览器自动化、Windows 自动化、ERP 操作或其他业务 Skill 已形成端到端闭环。
- 不应通过临时脚本或直接系统访问绕过 Gateway、File Service、权限和审计边界来补足该能力。

**当前处置**

- 仅按当前已发布的消息、路由、线程、分发、Hermes 集成、Context、响应和投递能力运行。
- 遇到需要 Skill Runtime 的业务请求时，明确标记为当前不支持，不伪造成功状态。

**后续方向**

Skill Runtime 的权限模型、确认机制、执行隔离、状态、重试和审计仍需后续设计、实现与验收。

### 3.4 Memory Runtime 未实现

**现状**

当前版本未实现 Memory Runtime，也未实现 automatic memory。Context 仅表示当前线程上下文，Timeline 是事实记录，Context Snapshot 是可重建的上下文缓存。

**运维影响**

- 不能承诺跨线程、跨会话的长期认知或自动偏好沉淀。
- Snapshot 保留或 Hermes 本地 Session 不能作为企业长期 Memory。
- 不得把消息归档、Timeline 或 Context Snapshot 宣传为 RAG、embedding、vector database 或 automatic memory。

**当前处置**

- 依据 Message Archive、Timeline 和当前线程权限构建 Context。
- Snapshot 异常时从权威事实重建，不从模型内部状态恢复企业事实。
- 详细术语边界遵循 [Context Runtime](../context/context-runtime.md)。

**后续方向**

长期 Memory 如进入实施，需要单独定义写入来源、纠错、删除、保留、权限和审计机制；当前没有实现承诺。

### 3.5 WDT / S6 未接入

**现状**

WDT（旺店通）与 S6 当前未接入 V2 Enterprise Runtime。

**运维影响**

- 当前版本不能从 WDT / S6 查询或写入真实业务数据。
- 订单、库存、物流、寄样、财务、线下业务或对账流程不能因 Gateway Runtime 已发布而标记为已完成。
- 不得使用脱离 Gateway 权限与审计边界的直连方式冒充正式集成。

**当前处置**

- 对 WDT / S6 相关请求明确返回或记录为当前未集成范围，避免生成虚假的业务成功结论。
- 部署与验收只覆盖当前发布模块，不把外部业务系统可达性加入已通过项目。

**后续方向**

接口能力、字段、权限、幂等、错误恢复和端到端验收均需在后续集成阶段确认；本文不预设实现方式。

### 3.6 Dispatch 与 Response 事务桥后续优化

**现状**

Dispatch Outbox、Dispatch Worker、Response Persistence 和 Delivery Worker 已属于当前发布能力，但 Dispatch 与 Response 之间的事务桥仍有后续优化空间。当前不得宣称从分发到响应持久化的全部阶段构成一个跨组件的单一原子事务。

**运维影响**

- 在进程退出、数据库异常、Hermes 调用异常或网络中断附近，需要分别核对归档、线程、Dispatch Outbox、响应持久化和投递状态。
- 不应仅凭入口已归档推断 Hermes 已完成，也不应仅凭 Hermes 返回推断响应已经持久化或投递成功。
- 投递失败不应重新触发 Hermes 推理；应从已持久化响应和当前投递状态继续排障。

**当前处置**

- 使用 Gateway 已持久化事实和当前实现提供的状态进行核对与恢复。
- 按数据库、migration、Gateway、Workers 的顺序恢复服务，再检查日志、health 和 heartbeat。
- 发现阶段间状态不一致时先保留证据，按发布实现的恢复机制处理，不直接修改数据库或伪造终态。

**后续方向**

后续可优化 Dispatch 与 Response 的事务衔接、一致性检查和恢复体验。具体事务模型、幂等契约和迁移方案尚未在本文确定。

## 4. 状态声明规则

- **已发布**只用于 `v2-enterprise-runtime-20260811` 已存在并可按当前实现运行的模块。
- **未实现**不得写为“已接入”“已支持”或“可用”。
- **未集成**不得因接口设想、Hermes 推理能力或手工操作而写成业务闭环。
- **测试 skip** 必须保留为验证缺口，不能计入通过数。
- **后续方向**不构成接口、时间或交付承诺。
- 当前限制清单不是新增功能清单；任何边界变化都应以代码、测试、部署证据和技术决策为依据。

## 5. 交接与验收检查

交接时至少核对：

- Windows 测试报告明确列出 symlink skip，并有 Debian 目标环境验证计划或结果。
- Hermes Integration 没有被描述为独立通用 remote transport。
- Skill Runtime 与 Memory Runtime 均明确标记为未实现。
- WDT / S6 均明确标记为未接入。
- Dispatch、Response Persistence 与 Delivery 的阶段状态可分别观察和排障。
- 文档没有把 RAG、embedding、vector database、automatic memory 或业务系统闭环写成当前能力。

整体模块边界见 [V2 Enterprise Runtime 架构总览](../architecture/v2-enterprise-runtime.md)，部署拓扑见 [Staging Debian 部署](../deployment/staging-debian.md)，运维步骤见 [Runtime 运维](../operations/runtime-operations.md)。
