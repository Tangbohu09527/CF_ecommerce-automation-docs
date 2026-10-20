# 企业 AI Gateway 架构

> 需求同步日期：2026-10-20；实现、合并、隔离测试和现场状态分别记录。本轮未访问生产。

## 定位

Gateway 是企业消息、身份、权限、线程、Context、Dispatch、Response、Delivery 和恢复审计的控制边界。PostgreSQL 是权威状态源，Hermes 是外部执行服务，agent-wechat 是外部通道服务。

业务链路为员工微信 → 微信入口 → Gateway 权限/上下文/任务 → 官方 Hermes 与工具 → 企业文件服务/后续业务系统 → 结果返回微信。员工与 AI 共用 FileBrowser 唯一正式文件系统。Gateway 管理授权、会话/Profile 路由、任务和必要管理状态；按[最新确认](../requirements/confirmed-platform-scope.md)先完成平台基础，商品别名、分类细节、手卡匹配、模板填充等后续交给 Hermes/Skills。普通文件按任务取得、按明确要求归档，见[文件业务需求](../requirements/file-workflows.md)；不得绕过 File Service API、权限和审计。

## 生产组件

| 组件 | 职责 |
| --- | --- |
| Gateway API | liveness/readiness、Runtime health、Admin 查询与恢复 |
| Poll Worker | WeChat Polling、Checkpoint、Persist-first、Admission、V2 Thread/Dispatch |
| Dispatch Worker | durable Dispatch、Hermes 调用、Response 持久化 |
| Delivery Worker | Outbox、Attempt/Receipt、微信投递、reconciliation |
| PostgreSQL | 全链路权威状态与 audit |
| Runtime Controller | v1 对 Poll Worker 与 Delivery Worker 执行组合 stop/start/status；不控制 Dispatch Worker |

以上是四个应用进程加 PostgreSQL。external agent-wechat 不属于 Gateway Compose 的应用进程。

## Git、候选与现场权威

当前入口使用[2026-10-20 总体固定证据](../validation/records/2026-10-20-overall-status-evidence.md)及[PR12 后续纯配置候选证据](../validation/records/2026-10-20-configuration-only-candidate.md)。最后实际现场观察为 10 月 19 日北京时间 18:36，本轮不是实时核验。10 月 11 日旧回退、候选和重启未知断点保留在[当日历史记录](../validation/records/2026-10-11-overall-status-evidence.md)，不再作为当前待执行步骤。

Source SHA、实际 Image ID、registry digest、完整构建来源、隔离测试、实机操作和业务验收分别留证。未合并候选不写成 main，已保留的官方 Hermes 纯配置实验不等于每任务可信授权关联、Artifact/Response/微信交付或新机证明全部完成；最终简部署目标后置，候选和失败事实均保留。

本轮必要只读代码核对使用 Gateway 固定 `91a235fca08362d8c1440e808df56946be3cc1e8`，文件/行及远端核对限制见[需求同步记录](../validation/records/2026-10-20-requirements-sync.md)。下列新增实现差距据此记录，不宣称远端最新或现场实时行为。

## V2 Thread

V2 RouteResolver 从已持久化 Conversation、Group Type、Profile revision 和 policy 解析路由，再由 ThreadResolver 构建 key。

```text
platform
+ source account
+ physical conversation
+ sender identity（group_sender/private_sender）
+ profile id/revision
+ thread policy
```

`group_shared` 明确排除 sender identity。Gateway 自动化测试覆盖：

- private sender isolation；
- group sender isolation；
- group shared reuse；
- profile revision 新线程；
- 并发首次解析幂等。

V1 compatibility key 仍以 source account + physical group conversation 建立 whole-room thread。生产 V2 不应回退到 V1；部署和排障必须核对 V2 Routing 开关。

这是已有实现。2026-10-20 已确认 B 方案是每员工/每微信群独立真实 Hermes Profile；分组只提供模板，私聊按员工组、群聊按群组选择并检查发起员工权限。每实体首次创建对应会话后持续绑定，`group_sender` 按发起者分线程以及 revision 改键与新目标存在差距；现有 `GroupType -> AgentProfile` 直接引用也不是模板派生关系。旧代码测试不等于新目标已完成。

## Profile 目标与权限

真实 Hermes Profile 包含独立配置、`SOUL.md`、Skills 和相应会话/记忆状态；不是提示词或会话 ID，也不是操作系统安全沙箱。先判断员工 Bot 使用权限，再判断群聊使用权限；授权可未分组，不能暗配默认档案。固定代码中通用用户策略和群准入已有基础，未见独立员工群聊使用权限字段，群未分组仍有 `unknown_group` 回退，与新要求需并列记录。分组不提升正式文件权限。

现有 Gateway 保存 `external_profile_ref`/revision，请求携带 Profile 引用/版本与 Hermes 会话信息，只证明已有引用和续接基础。D026 历史只引用边界调整为计划受控编排创建和同步，优先官方能力；实际路由、自动创建、修改同步、旧会话配置生效须分别验证。仅同步管理员明确管理的模板字段，不覆盖历史、记忆、运行状态或独立凭据；模板字段、Skills 来源和调组处理待细化。详见[最新权威确认](../requirements/confirmed-platform-scope.md)。

## Durable admission 与 Dispatch

- Message 先提交。
- Admission 有唯一 durable outcome。
- Allowed outcome 与 Dispatch enqueue 同事务完成。
- Dispatch 使用稳定 idempotency key、claim token、lease 和 fencing。
- `uncertain` 阻塞同线程后续工作并禁止自动重试。

当前同线程 FIFO 和 Hermes 会话续接不等于执行中补充。用户要求补充能够进入对应正在执行的过程，需区分当前任务补充、同会话独立新请求以及已结束/不可撤销后的修改；如何对接官方机制待验证，不预设复杂合批系统。

## Context Runtime

Context 只读取指定 AI Thread 中成功的 durable turn，支持 Timeline、recent、range、search 和 versioned Snapshot。授权 provider 绑定 enterprise identity 与 thread；跨线程读取 fail closed。

Context 能力已有 main 实现、测试及历史部署证据，但未逐项证明其所有 API/工具在生产被执行。Snapshot 不是 RAG 或长期 Memory。

Message 档案与成功任务 Timeline 不同。当前群近几条相关消息、明确引用和明确日期范围读取属于基础需求；现有 Context 工具只 join 成功 Dispatch/Response，尚不能保证取到已采集但未触发任务的群上文。人工 Web 与机器人应复用同一消息档案，机器人通过受控工具查询，不取得管理员 Token 或数据库权限。查询分页覆盖实际已存且获准记录；Poll 是采集，不是历史查询。self 跳过、Poll 停止/范围、上游窗口与连续性缺口保留，不能写成在线即全部已存。

## Admin recovery

受认证 Admin API 提供：

- Dispatch inspection；
- `retry-approved`；
- `mark-dead`；
- 证据支持的 `confirm-success`。

CAS 状态变更与 audit insert 在同一事务，恢复审计不可修改/删除。历史已有受控恢复记录，09-17 另有独立 `mark_dead` 审计证据；具体动作覆盖范围仍需继续演练。

## Response 与 Delivery

Dispatch Response、normalized Response、Delivery Outbox、Attempt、Receipt 和 reconciliation 分层保存。Response 缺 Delivery 是 reconciliation 问题，不是重新执行 Hermes 的理由。

文本响应与投递有历史实机证据，09-17 分钟级回传保留原日期。媒体候选包含持久入站取件、宿主授权和任务内附件返回代码及隔离验证；PDF/PNG 样本与正常 ZIP 出站已有真实限定验收，不能写成全部尚无实收，也不能扩大为通用 ZIP 入站或保存后独立检索交付均完成。

10-07 固定组件记录限定证明了两个指定 PDF/PNG 的文件 API 下载、官方 Hermes 内容读取与原聊天文字结果关联，保留当次 Skill 附带修改及模型传输原始图片字节未观测的边界。微信入站取得字节、内容理解、指定目录/分类归档、文件中心查询回传、原聊天实际附件实收及加工回存分别验收；HTTP 200、文本读取、READY 或 CI 绿不能替代最终实收。入站继续遵守 D027 受控持久化，出站继续遵守 D028 READY Artifact。

## Gateway Web 与分组分机：已确认规划

Web 规划七项：员工 Bot/群聊使用权限，员工/群分组，模板与派生 Profile/创建同步状态/版本/目标电脑，微信及采集/执行/投递状态，聊天档案按会话/人员/时间查询，AI 电脑/任务/实际资源，以及员工/群/组使用频率。现有数据与读取 API、缺管理写接口、缺资源/频率等采集、未有页面逐项记录于[权威正文](../requirements/confirmed-platform-scope.md)。现有心跳/auth/Hermes/任务指标不能替代 CPU/内存与使用频率采集，也不能称后台全部完成。

按部门、岗位或业务分组，再明确手工配置组到电脑，独立员工/群 Profile 部署在对应电脑。机器数由规模决定；不强制 UI 与非 UI 分机，不预设阈值分流、复杂均衡或执行中迁移。会话/任务/电脑映射及变更生效规则待细化，不能丢历史、重复执行或无声切换。权限仍在服务端，员工权限增删不固化为 SSH/SQL 手工流程。本轮不开发页面，原六阶段不变。

## Runtime health 与 P1

`/health` 是进程 liveness，`/ready` 是数据库/迁移 readiness，Runtime health 才反映 Worker heartbeat、Checkpoint continuity、Dispatch、reconciliation 和 Delivery 业务链。

Gateway P1 日志使用 `json-file 64m x 10`。agent-wechat 使用不同的 `20m x 3`，两者不得混用。

## 当前限制

- PostgreSQL restore 未演练。
- 同群多发送者旧隔离未单独生产验收；B 方案独立 Profile/群持续会话尚有实现差距。
- Context 全能力和 Admin 全动作未逐项生产演练。
- 通用 ZIP 入站、原文件名实际协议部署/收件验收、纯配置可信任务关联、新机/整机重启仍有缺口。
- 当前基础完成后才另行制作业务 Skills/独立仓库；不以新 Runtime/RAG、watchdog 或 HA 作为本轮条件。

更多信息见[系统设计](../02_系统设计.md)、[Recovery Runbook](../operations/recovery-runbook.md)和[当前状态矩阵](../status/current-status.md)。
