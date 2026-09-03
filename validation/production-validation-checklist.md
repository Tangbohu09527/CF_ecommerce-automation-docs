# Production Validation Checklist

> 中文名称：生产验证清单
>
> 文档编号：VAL-001
>
> 状态日期：2026-09-03

## A. 2026-09-03 已完成的生产验收摘要

本节只摘要已经完成的真实生产行为。详细版本、digest、回滚和证据边界见[Enterprise Runtime Production Closeout](./records/2026-09-03-enterprise-runtime-production-closeout.md)。

- [x] Gateway Git authority 为 merged main `b488cf452584e73bc9b752564bf90ea153aa8d18`；P1 未创建新 Tag。
- [x] Production image digest、Release label、database revision `20260823_04` 和 rollback Release 已记录。
- [x] PostgreSQL、Gateway API、Poll/Dispatch/Delivery Worker、external agent-wechat healthy。
- [x] Controller `ready=true` 且 `token_contract_valid=true`。
- [x] 获准私聊文本完成 Message -> Admission -> Dispatch -> Hermes -> Response -> Delivery -> 微信回复。
- [x] 真正结构化 `@` 的群聊文本完成实际回复。
- [x] 群聊未结构化 `@` 时不调用 AI。
- [x] Bot self reply 被跳过且无回复回环。
- [x] forced-QR 后 Local ID 回退和 Checkpoint generation 通过。
- [x] 历史前缀跳过，实时后缀只处理一次。
- [x] 私聊普通前进和空窗口实时后缀通过。
- [x] 无重复回复，Queue 与业务链一致，outstanding work 为零。
- [x] fresh QR、手机扫码、auth/chats/messages 和 Worker 重新放行通过。
- [x] CFserver reboot 后 Docker/存储/Gateway core 恢复，agent-wechat 保持停止；显式 stop Gate + fresh QR 后恢复在线。
- [x] AI host reboot 后 WeChat Session 保持，Hermes reachability 恢复，不需要 fresh QR。
- [x] Gateway-only cutover 未重建 agent-wechat，authenticated Session preserved。
- [x] P1 启动摘要和稳态日志降噪通过，未观察到目标重复日志或 ERROR/violation。
- [x] Gateway rollback Release、offline image archive、archive checksum 和 Evidence Run ID 已记录。

以下不属于上面已完成项：同群多发送者生产隔离、PostgreSQL restore、automatic boot stop gate、完整媒体、FileBrowser deployment、Skills、旺店通/S6、Hermes watchdog/HA。

## B. 未来 Release 可复用检查模板

每次执行复制为新的 `validation/records/<YYYY-MM-DD>-<release>-<scope>.md`，保留期望、实际、PASS/FAIL/BLOCKED/NOT_RUN 和脱敏证据。不要直接改写 A 节历史结果。

### B1. 发布输入

- [ ] 记录文档 commit。
- [ ] 记录 Gateway merged Git authority、production source SHA、image digest 和 revision。
- [ ] 记录 WeChat main/PR stack、批准 source、image digest 与构建映射证据。
- [ ] 记录 Hermes external runtime 控制和健康入口，不从历史文档猜版本。
- [ ] 记录 previous immutable rollback Release。
- [ ] 所有输入无漂移 `main`、`latest`、空占位符或 Secret。

### B2. 安全与备份

- [ ] PostgreSQL 备份存在，恢复目标和 restore 步骤明确。
- [ ] 若宣称 restore，通过隔离环境实际恢复并留证。
- [ ] agent-wechat Archive 与 Token 分离，Archive 不用于 active Session。
- [ ] Gateway 与 agent-wechat 日志策略分别为批准值。
- [ ] 证据不含账号、Chat ID、消息正文、QR、Token、Authorization、Cookie、database URL 或内网 IP。

### B3. Host 与部署

- [ ] CFserver/Windows Host 时间与同步符合策略。
- [ ] containers/PostgreSQL 为 UTC。
- [ ] 两个 Compose project ownership 明确。
- [ ] 未使用跨项目 `down` 或 `--remove-orphans`。
- [ ] PostgreSQL readiness 与 migration head 正确。
- [ ] Gateway API 和三个 Worker 使用批准 immutable image。
- [ ] Runtime Controller contract、status 和 Token contract 通过。

### B4. agent-wechat 与 forced QR

- [ ] `restart: "no"`、loopback 6174、`cf-internal`、`ENABLE_VNC=0` 和只读 Token mount。
- [ ] fresh QR 前显式 stop Gate。
- [ ] 旧 Runtime 归档但未复用。
- [ ] SSH TTY 实际显示 fresh QR；证据不保存 QR。
- [ ] WeChat process、container/API health、auth、chats、messages 全部通过。
- [ ] 所有门禁通过后才恢复 Workers。
- [ ] Host boot 到人工登录前的 automatic boot stop gate 已单独验证；否则记录 BLOCKED。

### B5. Gateway Runtime

- [ ] Poll Worker 从已有 Checkpoint/generation 继续，不重新 bootstrap。
- [ ] Dispatch Worker heartbeat fresh，Hermes reachability 真实检查通过。
- [ ] Delivery Worker heartbeat fresh，agent-wechat auth/API 可用。
- [ ] Runtime health 无未归属的 `uncertain`、stale lease、blocked thread、missing delivery 或 poison candidate。
- [ ] Queue backlog 和 oldest age 在批准阈值内。

### B6. 文本链

- [ ] 获准私聊只形成一个 Message、Admission、Dispatch、Response、Delivery 和可见回复。
- [ ] 未获准私聊持久化但不调用 Hermes。
- [ ] 群聊真实 `@` 完成回复。
- [ ] 群聊未 `@` 为 `bot_not_mentioned`，不调用 AI。
- [ ] 纯文本伪 `@`、引用或上一条 mention 不被继承。
- [ ] self reply 不进入 Message/Dispatch。
- [ ] 重复来源事件只命中同一逻辑链。

### B7. Thread 与 Context

- [ ] 私聊线程稳定。
- [ ] 两个获准发送者在同一群中使用不同 V2 `group_sender` AI Thread。
- [ ] V1 compatibility whole-room path 未被生产误启用。
- [ ] `group_shared` 未批准时不可用。
- [ ] Context read/recent/range/search 限定当前 identity/thread。
- [ ] Snapshot version 与 coverage 正确，未跨 unfinished turn。
- [ ] 引用正文只有在实际注入并验证后才能标记 PASS。

### B8. Admin recovery

- [ ] `uncertain` inspection 返回完整但脱敏证据。
- [ ] `retry-approved` 只在证明 Hermes 未执行时使用。
- [ ] `mark-dead` 不伪造 Response。
- [ ] `confirm-success` 要求匹配持久化证据。
- [ ] 并发恢复只有一个 CAS winner。
- [ ] recovery audit 不可 UPDATE/DELETE。
- [ ] 任何恢复都不使用 SQL 修改、Queue 清空或人工重发。

### B9. Response 与 Delivery

- [ ] Response 先持久化再创建 Outbox。
- [ ] Delivery 失败不产生第二次 Hermes Dispatch。
- [ ] Response 缺 Delivery 由 reconciliation 修复。
- [ ] ambiguous send 不盲目重发。
- [ ] 实际微信只收到一次正确会话回复。

### B10. Checkpoint 与恢复

- [ ] forced-QR Local ID 回退触发安全 generation rebase。
- [ ] historical prefix 不进入 Sink。
- [ ] live suffix 只处理一次。
- [ ] empty/unverified window fail closed。
- [ ] self reply 推进 Checkpoint。
- [ ] Worker restart 复用已有 Checkpoint。
- [ ] CFserver reboot、AI host reboot、Gateway-only deploy 和 agent-wechat restart 分别执行，不互相代替。

### B11. 媒体、文件和业务扩展

- [ ] Attachment 私有持久化、完整性和生命周期通过；未实现时 BLOCKED。
- [ ] Hermes 多模态协议通过；未实现时 BLOCKED。
- [ ] Artifact 原子物化、`READY` 和媒体 Delivery 通过；未实现时 BLOCKED。
- [ ] FileBrowser Candidate、migration、backup/restore、rollback、WebDAV/OnlyOffice 和权限审计通过；未部署时 BLOCKED。
- [ ] Skills 权限、确认、幂等和审计通过；未实现时 BLOCKED。
- [ ] 旺店通/S6 端到端业务验收通过；未接入时 BLOCKED。

### B12. 结果

- [ ] 记录 PASS、FAIL、BLOCKED、NOT_RUN 数量和原因。
- [ ] 记录消息丢失、重复、错投、权限和业务副作用判断。
- [ ] 记录 rollback 是否执行。
- [ ] 记录残余风险、owner 和复核时间。
- [ ] 最终决定为 GO、LIMITED-GO 或 NO-GO，并有签核。
