# Production Validation Checklist

> 中文名称：生产验证清单
>
> 文档编号：VAL-001
>
> 状态日期：2026-10-11；A 节保留 2026-09-03 历史结果，本次未执行生产命令或业务验收。

最新截止点见[2026-10-11 脱敏证据摘要](./records/2026-10-11-overall-status-evidence.md)：旧回退及旧核心恢复已完成，新候选和两端新 plan 已通过，但新 deployment 尚未 `apply/start/verify`。用户随后已重启 CFserver，重启后状态未核验，Windows 是否关机未确认。应先核验重启后现场与现存新计划，符合条件再接续切换；不能把重启前健康、旧 PID 或历史勾选当作本次放行依据，也不从头重复回退、传包或构建。

## A. 2026-09-03 已完成的生产验收摘要

本节只摘要已经完成的真实生产行为。详细版本、digest、回滚和证据边界见[Enterprise Runtime Production Closeout](./records/2026-09-03-enterprise-runtime-production-closeout.md)。

- [x] 2026-09-03 Gateway Production Release Git authority 为 `b488cf452584e73bc9b752564bf90ea153aa8d18`；P1 未创建新 Tag。
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
- [x] CFserver reboot 后 Docker/存储/Gateway core 恢复，agent-wechat 保持停止；Controller `stop` 组合 Poll/Delivery Gate + fresh QR 后恢复在线。
- [x] AI host reboot 后 WeChat Session 保持，Hermes reachability 恢复，不需要 fresh QR。
- [x] Gateway-only cutover 未重建 agent-wechat，authenticated Session preserved。
- [x] P1 启动摘要和稳态日志降噪通过，未观察到目标重复日志或 ERROR/violation。
- [x] Gateway rollback Release、offline image archive、archive checksum 和 Evidence Run ID 已记录。

以下不属于上面 2026-09-03 已完成项：同群多发送者生产隔离、PostgreSQL restore、automatic boot stop gate、完整媒体、FileBrowser deployment、Skills、旺店通/S6、Hermes watchdog/HA。FileBrowser 后续已有 09-25 至 09-27 限定部署、读取和文本新建证据，按[当前状态](../status/current-status.md)分别记录，不沿用历史待部署结论覆盖后续进展。

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
- [ ] Runtime Controller v1 contract 确认 controlled services 仅为 `worker` 与 `delivery-worker`，`stop/start` 始终组合控制二者。
- [ ] Controller status、Token contract 和 `ready=true` 通过；未假设 Poll-only、Delivery-only 或 Dispatch 控制。

### B4. agent-wechat 与 forced QR

- [ ] `restart: "no"`、loopback 6174、`cf-internal`、`ENABLE_VNC=0` 和只读 Token mount。
- [ ] fresh QR 前通过 Controller `stop` 显式关闭组合 Poll/Delivery Gate。
- [ ] 旧 Runtime 归档但未复用。
- [ ] SSH TTY 实际显示 fresh QR；证据不保存 QR。
- [ ] WeChat process、container/API health、auth、chats、messages 全部通过。
- [ ] 所有门禁通过后，才通过 Controller `start` 同时准备、重建并启动 Poll Worker 与 Delivery Worker。
- [ ] Host boot 到人工登录前的 automatic boot stop gate 已单独验证；否则记录 BLOCKED。

### B5. Gateway Runtime

- [ ] Poll Worker 从已有 Checkpoint/generation 继续，不重新 bootstrap。
- [ ] Dispatch Worker 由 Gateway Release/Compose 生命周期独立启动，heartbeat fresh，Hermes reachability 真实检查通过。
- [ ] Controller `start` 后 Poll Worker 与 Delivery Worker 均 healthy、heartbeat fresh，agent-wechat auth/API 可用。
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

沿用[9 月 27 日文件业务场景](../requirements/file-workflows.md)，以下六项必须独立给出期望、实际、状态和脱敏证据。当前交付先聚焦真实 PDF/图片收发，其余业务步骤按既有优先级接续，不并行扩安装器或大重构。下表是截至 2026-10-11 的证据边界，不能直接勾选为未来 Release 的 PASS。

| 独立验收项 | 通过标准 | 已有局部证据与当前缺口 |
| --- | --- | --- |
| 微信入站取得字节 | 指定获准消息取得可读取的真实文件/图片字节，记录类型、大小、摘要与源样本对比；Gateway 受控持久化和附件引用可核对。图片区分原图、标准图、缩略图与转码结果 | 历史 JPEG 读取有证据；9b 微信候选的固定制品及合成缓存隔离测试通过，不能证明已登录微信的原生下载。真实 PDF/图片完整入站尚未验收 |
| 内容理解 | Hermes/适用工具实际读取已取得的内容；用样本核对提取信息与分类依据，缺失或看不清的信息明确待确认，不猜造商品归属、报告字段或授权有效性 | 09-26 有既有文件读取，10-06 有已存 PDF/PNG 的官方 Hermes 本机理解，10-07 有文字取文件及文字回复的固定证据；均不能替代微信新入站同一原件的理解验收。格式与多模态支持分样本记录，完整场景未验收 |
| 指定目录/分类归档 | 群、身份、目录规则已授权；内容分类经校验后通过 FileBrowser API 保存到指定目录，核验实际字节、去重、来源关联及审计；未知/冲突进入待确认，同名不同内容不静默覆盖 | 09-27 受控文本新建有证据；真实共享群附件收妥、分类和正式归档未验收。常规预授权新增自动执行仍是目标，不能为验收而全局放行 |
| 文件中心查询回传 | 按请求人/会话权限检索唯一正式 FileBrowser 文件系统，核对选中文件、取得字节、Gateway READY Artifact 与投递关联；只分发原件不强制经过 AI 主机加工 | 09-26 读取、10-07 File API 取件与原聊天文字回复及客户端基础可复用；完整“检索选件→受控物化→附件投递”未验收。读到内容或返回路径/链接不能代替附件回传 |
| 原聊天实际附件实收 | 原请求聊天实际收到且可打开正确文件/图片；核对样本身份、内容完整性和单次投递，文件发送与图片消息分别留证；不错投、不重复发送 | 09-17 文本长任务回传仅证明文本场景；HTTP 200、READY 或发送接口成功不等于收件端实收。10-11 新 deployment 尚未 `apply/start/verify`，附件实收未验收 |
| 加工结果回存 | 经 API 下载工作副本到隔离任务目录，工具加工后检查产物，再经 FileBrowser API 新建文件/新版并回读核验，记录原件关联、权限和审计；默认保留原件 | 独立客户端有部分基础；完整 Agent 下载、加工、产物回存及原件关联未验收。文本新建不能代替此项，也不证明完整版本管理 |

- [ ] 六项分别填写 PASS/FAIL/BLOCKED/NOT_RUN；文本新建、读取成功、HTTP 200、CI 绿不能互相替代，附件元数据不等于原件。
- [ ] Attachment 私有持久化、完整性、生命周期与 AI 主机离线后的续接分别留证；未取得原件不称“已收妥”。
- [ ] 出站 Artifact 原子物化、`READY`、Delivery 回执与收件端实收分别核对，不因投递不确定盲目重发。
- [ ] FileBrowser 部署、migration、backup/restore、rollback、WebDAV/OnlyOffice 和权限审计逐项记录；既有局部部署/客户端审计不代替完整发布或服务端 Persistent Audit。
- [ ] Skills 权限、确认、幂等和审计通过；未实现时 BLOCKED。
- [ ] 旺店通/S6 端到端业务验收通过；未接入时 BLOCKED。

### B12. 结果

- [ ] 记录 PASS、FAIL、BLOCKED、NOT_RUN 数量和原因。
- [ ] 记录消息丢失、重复、错投、权限和业务副作用判断。
- [ ] 记录 rollback 是否执行。
- [ ] 记录残余风险、owner 和复核时间。
- [ ] 最终决定为 GO、LIMITED-GO 或 NO-GO，并有签核。
