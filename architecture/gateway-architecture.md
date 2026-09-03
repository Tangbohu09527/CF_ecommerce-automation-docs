# 企业 AI Gateway 架构

> 状态日期：2026-09-03

## 定位

Gateway 是企业消息、身份、权限、线程、Context、Dispatch、Response、Delivery 和恢复审计的控制边界。PostgreSQL 是权威状态源，Hermes 是外部执行服务，agent-wechat 是外部通道服务。

## 生产组件

| 组件 | 职责 |
| --- | --- |
| Gateway API | liveness/readiness、Runtime health、Admin 查询与恢复 |
| Poll Worker | WeChat Polling、Checkpoint、Persist-first、Admission、V2 Thread/Dispatch |
| Dispatch Worker | durable Dispatch、Hermes 调用、Response 持久化 |
| Delivery Worker | Outbox、Attempt/Receipt、微信投递、reconciliation |
| PostgreSQL | 全链路权威状态与 audit |
| Runtime Controller | Poll/Delivery Gate 的 stop/start/status |

以上是四个应用进程加 PostgreSQL。external agent-wechat 不属于 Gateway Compose 的应用进程。

## Git 与发布权威

- Gateway `main`：`b488cf452584e73bc9b752564bf90ea153aa8d18`。
- Production-validated source：`f36c798294368263433f6132366ac9a864d9482b`。
- Database revision：`20260823_04`。
- Release：`p1-observability-main-b488cf452584-20260903`。
- Git authority 是 merged main SHA，不是历史 Tag。

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

Context 能力已实现、测试并进入 Gateway main/部署代码，但本次 Closeout 未逐项证明其所有 API/工具在生产被执行。Snapshot 不是 RAG 或长期 Memory。

## Admin recovery

受认证 Admin API 提供：

- Dispatch inspection；
- `retry-approved`；
- `mark-dead`；
- 证据支持的 `confirm-success`。

CAS 状态变更与 audit insert 在同一事务，恢复审计不可修改/删除。生产已有一次受控恢复成功；具体动作覆盖范围仍需继续演练。

## Response 与 Delivery

Dispatch Response、normalized Response、Delivery Outbox、Attempt、Receipt 和 reconciliation 分层保存。Response 缺 Delivery 是 reconciliation 问题，不是重新执行 Hermes 的理由。

当前文本响应与投递已验证。完整媒体链仍缺 Attachment、Hermes 多模态、Artifact 原子物化和微信媒体接收证据。

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
