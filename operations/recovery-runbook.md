# Recovery Runbook

> 中文名称：跨系统生产故障恢复手册
>
> 文档编号：OPS-001
>
> 状态日期：2026-09-03

## 1. 固定流程

每个场景统一执行：

1. **检查：** 保存版本、digest、revision、健康、heartbeat、backlog 和脱敏关联 ID。
2. **分类：** 确认所有者和第一个不满足不变量的节点。
3. **操作：** 只执行目标组件已批准的最小恢复动作。
4. **验证：** 核对 Message -> Admission -> Dispatch -> Response -> Delivery，不以单一 health 替代。
5. **回滚：** 使用 previous immutable Release 或组件批准的停止流程。
6. **保存证据：** 保留 UTC 时间、操作、结果、日志引用和残余风险，不保存 Secret/正文。

## 2. 所有者

| 所有者 | 负责范围 |
| --- | --- |
| Operator | 事故记录、Gate、变更顺序、验证和证据 |
| `CF_agent-gateway` | API、Poll/Dispatch/Delivery、Controller、Context、Admin recovery |
| PostgreSQL | revision、权威状态、备份与 restore |
| `CF_agent-wechat` | container、WeChat process、fresh QR、auth/chats/messages |
| Hermes / AI host | external runtime、reachability、watchdog、capacity |
| FileBrowser | future File Service deployment and recovery |

## 3. 禁止事项

- 不删除数据库行、Checkpoint、Queue、Response、Outbox 或 recovery audit。
- 不手工改 Checkpoint、伪造 Response 或直接重发结果。
- 不自动复用旧 WeChat Session 或 Archive。
- 不覆盖 active Release，不使用漂移镜像。
- 不跨项目执行 Compose `down` 或 `--remove-orphans`。
- 不打印 Token、Authorization、Cookie、QR、database URL、账号、Chat ID 或消息正文。
- 不在保存证据前删除容器或日志。

## 4. 整体机器人无回复

**Owner:** Operator，按首个失败节点转交 Gateway / WeChat / Hermes。

- **检查：** Controller status、agent-wechat auth/chats/messages、三个 Worker heartbeat、Runtime health、`uncertain` 和 backlog。
- **分类：** 入口未读、Admission 预期拒绝、Dispatch、Hermes、Response 或 Delivery。
- **操作：** 从入口到投递逐段定位，只恢复第一个失败节点。
- **验证：** 使用一条获准脱敏文本串联完整 durable chain 和微信实际回复。
- **回滚：** 若由新 Release 引起，转 previous immutable Release。
- **证据：** 保存关联 ID、各阶段状态和首个失败点。

## 5. Controller `ready=false`

**Owner:** `CF_agent-gateway` + Operator。

- **检查：** database/revision、Controller contract、Token contract、目标 Worker health。
- **分类：** schema、配置、Token contract、Worker 或外部 WeChat。
- **操作：** 保持 Gate 关闭，修复明确前置条件后重新 status。
- **验证：** `ready=true`、`token_contract_valid=true`，无异常 backlog。
- **回滚：** 回到 previous immutable Gateway Release。
- **证据：** 保存 Controller 脱敏输出、revision 和 Release。

## 6. Poll Worker stopped

**Owner:** `CF_agent-gateway`。

- **检查：** heartbeat、容器状态、Checkpoint continuity、agent-wechat auth。
- **分类：** 正常 Gate stop、进程失败、heartbeat 写失败或依赖失败。
- **操作：** 只有 WeChat/API/DB 正常且 Gate 允许时，按 Controller 恢复 Poll Worker。
- **验证：** heartbeat fresh；从原 Checkpoint 继续；不重新 bootstrap。
- **回滚：** 再次 stop Gate。
- **证据：** 保存前后 Checkpoint generation、heartbeat 和日志。

## 7. Delivery Worker stopped

**Owner:** `CF_agent-gateway`。

- **检查：** Response、Outbox、Attempt/Receipt、agent-wechat auth 和 heartbeat。
- **分类：** 正常 Gate stop、进程失败、通道不可用或 ambiguous send。
- **操作：** 已有 Response 时只恢复 Delivery；ambiguous send 不盲重发。
- **验证：** backlog 下降且没有第二次 Hermes Dispatch 或重复可见回复。
- **回滚：** stop Delivery Gate。
- **证据：** 保存 Outbox/Attempt 状态和实际接收结果。

## 8. Dispatch pending/running/uncertain

**Owner:** `CF_agent-gateway` + Hermes owner。

- **检查：** Admin inspection、lease、Hermes evidence、Response、Delivery 和同线程后续阻塞。
- **分类：** 正常 queued/running、stale lease、definite failure 或 `uncertain`。
- **操作：** `uncertain` 仅使用 `retry-approved`、`mark-dead` 或 evidence-backed `confirm-success`；禁止 SQL 修改。
- **验证：** 恢复 audit 存在，状态转换合法，后续队列符合 FIFO。
- **回滚：** 无法证明安全时保持 `uncertain` 或 `mark-dead`，不强制重做。
- **证据：** 保存 inspection、外部执行证据、operator/reference/reason 和 audit ID。

## 9. Hermes unreachable

**Owner:** Hermes / AI host。

- **检查：** AI 主机状态、Hermes process/health、CFserver reachability、最近 reboot 和 Dispatch 状态。
- **分类：** 主机、进程、网络、配置、凭证或容量。
- **操作：** 按 AI host 批准方式恢复 Hermes；先阻止新的危险 Dispatch。
- **验证：** reachability、一次受控文本调用、Response/Delivery 和 Queue。
- **回滚：** 无法稳定恢复时保持 Dispatch stopped，并回退最近 Hermes 变更。
- **证据：** 保存脱敏健康、时间和受影响 Dispatch 分类。

## 10. agent-wechat stopped

**Owner:** `CF_agent-wechat` + Operator。

- **检查：** 容器、`restart: "no"`、Gate、Runtime/Archive 目录元数据和批准 image。
- **分类：** Host/container/Runtime restart，或受控停止。
- **操作：** 显式 stop Gate，使用唯一 forced-QR 启动入口；不得直接 Compose `up`/restart 复用 Session。
- **验证：** process、container/API health、auth/chats/messages、Controller。
- **回滚：** 保持 Gate 关闭并停止 agent-wechat。
- **证据：** 保存阶段和 aggregate pass/fail，不保存 QR/Session payload。

## 11. agent-wechat `logged_out`

**Owner:** `CF_agent-wechat` + Operator。

- **检查：** WeChat process、auth、chats/messages、Gate 和上游可读窗口。
- **分类：** Session 失效，不按网络故障处理。
- **操作：** stop Gate，归档旧 Runtime，fresh QR。
- **验证：** auth/chats/messages 全部通过后才 start Gate。
- **回滚：** 登录失败时 agent-wechat 与 Workers保持停止。
- **证据：** 保存时间、image、流程阶段和 API aggregate 结果。

## 12. CFserver reboot

**Owner:** Operator；各组件 owner 配合。

- **检查：** Docker、storage、PostgreSQL、Gateway、Controller、agent-wechat 和 Worker 实际状态。
- **分类：** 核心恢复、agent-wechat 预期停止、Gate 是否意外运行。
- **操作：** 不假设 automatic boot stop；先显式 stop Gate，再 fresh QR，最后恢复 Workers。
- **验证：** revision、Controller、auth/chats/messages、Hermes reachability、Queue 和文本闭环。
- **回滚：** 任一门禁失败时保持 Gate 关闭。
- **证据：** 保存 boot 时间、服务状态、Gate 纠正动作和最终验证。

## 13. AI host reboot

**Owner:** Hermes / AI host。

- **检查：** CFserver/agent-wechat 是否保持、Hermes process 和 reachability。
- **分类：** 单纯 AI host reboot，不触发 WeChat fresh QR。
- **操作：** 按批准方式恢复 Hermes。
- **验证：** Controller ready、Hermes reachability、Queue、一次受控文本调用。
- **回滚：** 保持 Dispatch stopped，回退 AI host 最近变更。
- **证据：** 保存 reboot 后恢复时间和健康；不宣称 watchdog/HA 超出证据。

## 14. Gateway-only failed deployment

**Owner:** `CF_agent-gateway` + Operator。

- **检查：** new/previous SHA、digest、revision、migration、agent-wechat container identity 和 Session。
- **分类：** application、schema、Controller、Worker 或 health failure。
- **操作：** 不重建 agent-wechat；停止受影响 Gateway Worker，回退 previous immutable Release。
- **验证：** local Release、revision compatibility、Controller、Session preserved 和文本闭环。
- **回滚：** 使用已记录 rollback Release；数据库另行判断。
- **证据：** 保存前后 digest、rollback result 和 agent-wechat 未重建证明。

## 15. Checkpoint continuity warning

**Owner:** `CF_agent-gateway`。

- **检查：** generation、anchor/fingerprint、visible bounds、failure code 和是否 forced QR。
- **分类：** safe rebase、legacy anchor enrollment、empty window 或 unverified continuity。
- **操作：** 让当前 Runtime CAS/fail-closed 逻辑处理；不手工降低 Checkpoint。
- **验证：** historical prefix 未进入 Sink，live suffix 只处理一次，无 duplicate reply。
- **回滚：** 无法证明连续性时保持该 Chat stopped/degraded。
- **证据：** 保存 hashed scope、generation 和 aggregate counters。

## 16. duplicate/replay suspicion

**Owner:** `CF_agent-gateway`。

- **检查：** source identity、Message uniqueness、Admission outcome、Dispatch idempotency、Response 和 Delivery receipt。
- **分类：** 重读被幂等吸收、重复 Dispatch、重复 Delivery 或仅重复日志。
- **操作：** 立即停止相关 Gate/Worker，保留所有事实；不删除重复记录。
- **验证：** 一个来源事实只对应一个逻辑 Dispatch 和一个可见回复。
- **回滚：** 回退引入重复的 Release，保留数据库用于调查。
- **证据：** 保存关联链和重复发生层级。

## 17. database revision mismatch

**Owner:** PostgreSQL + `CF_agent-gateway`。

- **检查：** current/head revision、Release、startup check 和备份。
- **分类：** 未迁移、部分迁移、错误镜像或错误数据库。
- **操作：** 停止全部应用进程，在独占窗口运行批准 migration；不改版本表。
- **验证：** revision `20260823_04` 或目标 Release head、readiness 和关键不变量。
- **回滚：** 按 schema compatibility 选择应用回退或完整数据库 restore。
- **证据：** 保存 migration 输出、前后 revision 和备份引用。

## 18. queue backlog

**Owner:** `CF_agent-gateway`；依赖 owner 配合。

- **检查：** oldest age、queued/running/failed/uncertain、blocked thread、reconciliation poison 和 Worker heartbeat。
- **分类：** 容量、依赖、`uncertain`、poison candidate 或 Delivery outage。
- **操作：** 修复首个依赖；逐项使用现有恢复能力，不清空 Queue。
- **验证：** backlog 稳定下降，新增消息可完成且无重复。
- **回滚：** 停止新增流量并回退相关 Release。
- **证据：** 保存前后 aggregate counts 和 oldest age。

## 19. FileBrowser unavailable

**Owner:** FileBrowser。

- **检查：** 当前是否已正式部署和接入；目前系统状态为 future boundary。
- **分类：** 未部署时不是当前文本 Runtime 事故。
- **操作：** 不让 Gateway/Hermes 直连存储绕过 File Service。
- **验证：** 未来按独立 health、permission、audit、backup/restore 清单。
- **回滚：** 关闭未来文件集成，保持文本 Runtime 独立。
- **证据：** 保存 Candidate、migration、client integration 和 restore 记录。

## 20. rollback to previous immutable Release

**Owner:** Operator + 目标组件 owner。

- **检查：** previous digest、schema compatibility、rollback Release、日志和 outstanding work。
- **分类：** 应用回滚、数据库 restore、agent-wechat 回滚分别处理。
- **操作：** stop affected flow，切换到 previous immutable Release；agent-wechat 变更后仍 fresh QR。
- **验证：** digest、revision、Controller、Session规则、Queue 和生产清单。
- **回滚：** 回滚自身失败时保持 Gate 关闭并升级。
- **证据：** 保存 before/after Release、digest 和验证结果。

## 21. preserve logs before container recreation

**Owner:** Operator。

- **检查：** 目标容器、日志 driver、时间范围、剩余磁盘和敏感字段风险。
- **分类：** Gateway `64m x 10` 或 agent-wechat `20m x 3`。
- **操作：** 在受控位置保存必要日志与 inspect 结果；不复制 Secret、正文、QR 或 Session。
- **验证：** 证据可读、时间为 UTC、组件/Release/关联 ID 可定位。
- **回滚：** 证据不完整时暂停 recreation。
- **证据：** 记录保存位置的受控引用与校验，不把敏感原文提交 Git。

## 22. 事故关闭

只有根因、影响、版本、全部操作、消息丢失/重复判断、权限影响、回滚、验证和残余 owner 都记录后才能关闭。新运行事实写入 `validation/records/`；旧记录不改写。
