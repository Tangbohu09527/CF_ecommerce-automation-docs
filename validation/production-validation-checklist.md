# Production Validation Checklist

> 中文名称：生产验证清单
>
> 文档编号：VAL-001
>
> 状态日期：2026-10-20；A 节保留 2026-09-03 历史结果，本次未执行生产命令或业务验收。

最新证据见[2026-10-20 跨仓摘要](./records/2026-10-20-overall-status-evidence.md)：PDF、PNG 及正常商品 ZIP 出站已有限定验收；最后现场观察是 **2026-10-19 北京时间 18:36**，收发关闭、无在途、原 Bot 登录，随后临时授权撤销、临时密码删除。本轮未重新核验现场。10 月 11 日旧回退/新计划未切换只保留为历史，不能继续列为当前待执行步骤。B 节为未来 Release 模板，历史勾选不构成新放行依据。

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
- [ ] 宿主、应用/日志与数据库存储/连接时区按批准策略分别核对；时区候选构建不代替现场配置验收。
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

沿用[9 月 27 日文件业务场景](../requirements/file-workflows.md)，六项独立记录期望、实际、状态和脱敏证据。下表为本轮可核对的证据边界，不能直接勾选为未来 Release 的 PASS。

| 独立验收项 | 通过标准 | 现有结果与限制 |
| --- | --- | --- |
| 微信入站取得字节 | 获准消息实际取得可读取文件/图片，受控持久化与附件引用可核对；企业原始资料、ZIP 原件及成员核验完整性 | PDF 实际正文/附件/摘要一致；PNG 按当前业务口径已验收。ZIP 入站仍未完整验收；历史超时及一次失败说明投递保留，不判为损坏或格式不支持 |
| 内容理解 | 真实模型/适用工具实际读取内容，描述与样本一致；看不清/缺失信息待确认，不猜造商品归属、报告或授权事实 | PDF 正文与 PNG 实际读取、准确描述通过；后续文字同会话接续；不推广为所有格式、多轮/多人或复杂续办通过 |
| 指定目录/分类归档 | 在获准范围通过 FileBrowser API 归档，核对分类、字节、来源和审计，未知/冲突待确认，同名不同内容不静默覆盖 | 一个商品 4 主图、8 详情成功，原包保留；首次导入失败与后续新操作成功分列。非全企业资料库；共享群批量准入、去重/冲突/离线续接待规则与独立验收，路径调整明确暂缓 |
| 文件中心查询回传 | 按身份/会话权限查询唯一 File Service，实际选件、导出、READY Artifact/Delivery 可关联；不强制原件绕行 AI 主机 | 正常微信商品 ZIP 已由正式模型查询/导出新包并交付；受限 API 查询/复制/移动/改名/替换/可恢复删除/还原有证据，不等于每项均由真实模型选择 |
| 原聊天实际附件实收 | 原请求会话收到且可打开正确文件/图片；企业原始资料与 ZIP 核对自身导出/Artifact/收件整包和成员，确认不错投/不重复 | PDF、PNG 微信图片和正常 ZIP 出站实收已验证。**PNG图片识别及微信图片收发已验收，满足当前业务要求。** 原 PNG 字节差异、重编码和单独 file 附件路由不再列为待修阻塞；变化原因未定位。ZIP 18,167,063 字节、12 成员一致；文件名前缀/中文替换未通过原名验收，候选未生产验证 |
| 加工结果回存 | API 下载任务工作副本、加工/检查，再另存新文件或新版并回读、关联原件及审计，默认保留原件 | 客户端和素材归档/导出可复用；通用 Agent 加工回存、原件关联和完整版本管理仍需独立业务验收 |

- [ ] 六项分别写 PASS/FAIL/BLOCKED/NOT_RUN；成功文件操作不代替真实模型选用，单商品不代替全库，ZIP 出站不代替 ZIP 入站。
- [ ] Attachment 持久化、生命周期、AI 主机离线续接分别留证；元数据不等于已收妥原件。
- [ ] READY Artifact、Delivery 与收件分别核对；不因投递不确定盲目重发，不重放成功操作补证。
- [ ] FileBrowser migration、backup/restore、rollback、WebDAV/OnlyOffice、权限/服务端审计关联逐项验收。
- [ ] 首批文件类 Skill 明确已有工具、业务规则与缺现场验收；可小批量试点，不要求先建设新 RAG/自研 Runtime，不因工具安装直接标记 Skill 已交付。
- [ ] 旺店通/S6 未接入部分保持 BLOCKED；不以文件链通过替代 ERP 业务。

最终部署标准另外验收，不用工具发现或既有兼容 CI 代替：

- [ ] 服务端一条正式可复用入口，必要参数完整，新环境可独立部署/恢复，组件 ownership 清楚。
- [ ] 电脑仅官方 Hermes 加少量配置，无额外项目 EXE、手工复制插件、源码修改或项目专用常驻进程。
- [ ] 不依赖 Codex 工作树、`.task-artifacts`、旧现场回执；正式业务凭据与临时运维权限分离。
- [ ] 临时运维权限撤销后，业务凭正式凭据实际继续运行并留证；撤销成功不等于此项通过。
- [ ] MCP 如被采用，分别验证发现、权限/撤销、真实文件工具、Artifact/Delivery 和完整业务；隔离发现成功不能宣布生产迁移。

当前插件/客户端桥接状态与最终目标分列，统一入口/官方 Hermes 纯配置接入开发验证中。完整条件见[部署指南](../deployment/deployment-guide.md#最终部署标准与候选边界)。

项目最后三项另行验收：群聊及多人/连续交互；AI 按当前场景/问题/任务主动检索获准的完整聊天档案（不限 Hermes 历史/成功任务，不等于手机微信所有历史已采集）；向获准人员/群聊群发消息及附件。现有 Context 或真正 @ 文本测试不替代这些需求。Gateway Web 仍为后续规划。

### B12. 结果

- [ ] 记录 PASS、FAIL、BLOCKED、NOT_RUN 数量和原因。
- [ ] 记录消息丢失、重复、错投、权限和业务副作用判断。
- [ ] 记录 rollback 是否执行。
- [ ] 记录残余风险、owner 和复核时间。
- [ ] 最终决定为 GO、LIMITED-GO 或 NO-GO，并有签核。
