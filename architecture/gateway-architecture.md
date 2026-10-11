# 企业 AI Gateway 架构

> 状态同步日期：2026-10-11；实现、合并、隔离测试和现场状态分别记录。

## 定位

Gateway 是企业消息、身份、权限、线程、Context、Dispatch、Response、Delivery 和恢复审计的控制边界。PostgreSQL 是权威状态源，Hermes 是外部执行服务，agent-wechat 是外部通道服务。

业务链路为员工微信 → 微信入口 → Gateway 权限/上下文/任务 → 官方 Hermes 与工具 → 企业文件服务/后续业务系统 → 结果返回微信。员工与 AI 共用 FileBrowser 唯一正式文件系统；内容分类、指定目录归档、查找回传、工作副本加工后 API 回存沿用[文件业务需求](../requirements/file-workflows.md)。Gateway 管理授权、引用与任务状态，不能通过挂载正式目录替代 FileBrowser 的 API、权限和审计。

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

- 2026-10-11 GitHub 只读核对 main 为 `7963bc0db5a01e38099e021959feabf648db7ede`，包含 09-17 文档 PR #12；live tip 仍须按查询截止点理解。
- 固定修复 `d9bae61a043f1be4303e5c23661ec51904d5e708` 位于 `codex/fix-hermes-env-loopback`，尚未合并 main。它是 Draft/Open [PR #14](https://github.com/Tangbohu09527/CF_agent-gateway/pull/14) 的 f9 head 之后两个提交，修复分支没有关联 PR，不能把 d9 写成 PR #14 当前 head。
- d9 的 CI、官方 Hermes 兼容与隔离安装测试均通过；包含持久 HOST、官方启动 TLS 兼容、原 baseline 身份及中断回退收尾修复。测试环境不代表 CFserver 已启用新返回链路。
- 用户 10-11 回传旧回退/旧核心恢复和新候选/两端 plan 已完成，但新 deployment 未 `apply/start/verify`。随后 CFserver 已重启，重启后未核验；当前部署状态不能由源码或重启前健康推定。
- 09-03 P1 source、Release、image 和 schema，及 09-17 长任务镜像均作为各自日期历史证据保留，不能继续写成 10-11 当前生产权威。d9 CI 的 schema head 与现场实际 revision 也不能互相代替。

完整 SHA、PR、CI 与用户现场来源统一见[10 月 11 日摘要](../validation/records/2026-10-11-overall-status-evidence.md)。Source SHA、实际 Image ID、registry digest、完整构建来源、实机操作和业务验收分别留证；本次未创建生产 Tag。

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

## Durable admission 与 Dispatch

- Message 先提交。
- Admission 有唯一 durable outcome。
- Allowed outcome 与 Dispatch enqueue 同事务完成。
- Dispatch 使用稳定 idempotency key、claim token、lease 和 fencing。
- `uncertain` 阻塞同线程后续工作并禁止自动重试。

## Context Runtime

Context 只读取指定 AI Thread 中成功的 durable turn，支持 Timeline、recent、range、search 和 versioned Snapshot。授权 provider 绑定 enterprise identity 与 thread；跨线程读取 fail closed。

Context 能力已有 main 实现、测试及历史部署证据，但未逐项证明其所有 API/工具在生产被执行。Snapshot 不是 RAG 或长期 Memory。

## Admin recovery

受认证 Admin API 提供：

- Dispatch inspection；
- `retry-approved`；
- `mark-dead`；
- 证据支持的 `confirm-success`。

CAS 状态变更与 audit insert 在同一事务，恢复审计不可修改/删除。历史已有受控恢复记录，09-17 另有独立 `mark_dead` 审计证据；具体动作覆盖范围仍需继续演练。

## Response 与 Delivery

Dispatch Response、normalized Response、Delivery Outbox、Attempt、Receipt 和 reconciliation 分层保存。Response 缺 Delivery 是 reconciliation 问题，不是重新执行 Hermes 的理由。

文本响应与投递有历史实机证据，09-17 分钟级回传保留原日期。未合并媒体候选已包含持久入站取件、宿主授权和任务内附件返回代码及隔离验证，不能再将这些全部写为尚无实现；也不能据此宣称生产启用或微信附件实收通过。

10-07 固定组件记录限定证明了两个指定 PDF/PNG 的文件 API 下载、官方 Hermes 内容读取与原聊天文字结果关联，保留当次 Skill 附带修改及模型传输原始图片字节未观测的边界。微信入站取得字节、内容理解、指定目录/分类归档、文件中心查询回传、原聊天实际附件实收及加工回存分别验收；HTTP 200、文本读取、READY 或 CI 绿不能替代最终实收。入站继续遵守 D027 受控持久化，出站继续遵守 D028 READY Artifact。

## 轻量 Gateway Web 待规划

用户后续考虑局域网 IP 访问、Gateway 权限管理，以及查询 Gateway 已经接收的部分聊天/任务记录。这里只记录待规划范围，不开发、不承诺微信完整历史，不复制 FileBrowser 文件权限体系。旺店通/S6 等仍按原六阶段和已有证据作为后续业务集成。

## Runtime health 与 P1

`/health` 是进程 liveness，`/ready` 是数据库/迁移 readiness，Runtime health 才反映 Worker heartbeat、Checkpoint continuity、Dispatch、reconciliation 和 Delivery 业务链。

Gateway P1 日志使用 `json-file 64m x 10`。agent-wechat 使用不同的 `20m x 3`，两者不得混用。

## 当前限制

- PostgreSQL restore 未演练。
- 同群多发送者隔离未单独生产验收。
- Context 全能力和 Admin 全动作未逐项生产演练。
- 完整媒体/文件、Skills、Provider routing 和业务系统未完成。
- Hermes watchdog、告警、容量和高可用未完整收口。

更多信息见[系统设计](../02_系统设计.md)、[Recovery Runbook](../operations/recovery-runbook.md)和[当前状态矩阵](../status/current-status.md)。
