# Production Validation Checklist

> 中文名称：生产验证清单
>
> 文档编号：VAL-001
>
> 状态日期：2026-10-20；A 节保留 2026-09-03 历史结果，本次未执行生产命令或业务验收。

最新证据见[2026-10-20 跨仓摘要](./records/2026-10-20-overall-status-evidence.md)：PDF、PNG 及正常商品 ZIP 出站已有限定验收；最后现场观察是 **2026-10-19 北京时间 18:36**，收发关闭、无在途、原 Bot 登录，随后临时授权撤销、临时密码删除。本轮未重新核验现场。10 月 11 日旧回退/新计划未切换只保留为历史，不能继续列为当前待执行步骤。B 节为未来 Release 模板，历史勾选不构成新放行依据。

本轮仅完成最新需求与文档同步，未运行产品测试、安装、模型调用、Profile 创建/更新、生产连接或现场验收。以下新目标均不因文档确认变成 PASS。[最新权威需求与差异表](../requirements/confirmed-platform-scope.md)区分用户已确认、建议、待技术验证与已有实现；文档仓没有 workflow，文档检查不写成产品 CI 全绿。PR12 的[统一入口与官方 Hermes 最终隔离证据](./records/2026-10-20-configuration-only-candidate.md)保留，不能替代现场验收。

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

每次执行复制为新的 `validation/records/<YYYY-MM-DD>-<release>-<scope>.md`，保留期望、实际、PASS/FAIL/BLOCKED/NOT_RUN 和脱敏证据。不要直接改写 A 节历史结果。先标明验证现存契约还是新目标：旧 `group_sender`、self 跳过及 forced-QR 的历史成功不等于 B 方案、完整采集或在线保留登录已完成。

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

现存旧契约仍有 forced-QR 要求；最新恢复目标是先检查现有登录，在线保持、离线提示扫码。两者冲突尚未改脚本；下列旧入口项目只在其适用 Release 记录，不将每次强制扫码作为新恢复目标的通过标准。

- [ ] `restart: "no"`、loopback 6174、`cf-internal`、`ENABLE_VNC=0` 和只读 Token mount。
- [ ] fresh QR 前通过 Controller `stop` 显式关闭组合 Poll/Delivery Gate。
- [ ] 旧 Runtime 归档但未复用。
- [ ] SSH TTY 实际显示 fresh QR；证据不保存 QR。
- [ ] WeChat process、container/API health、auth、chats、messages 全部通过。
- [ ] 所有门禁通过后，才通过 Controller `start` 同时准备、重建并启动 Poll Worker 与 Delivery Worker。
- [ ] Host boot 到人工登录前的 automatic boot stop gate 已单独验证；否则记录 BLOCKED。
- [ ] 后续新恢复入口单独验证：检查并恢复缺失服务，在线微信保持登录、离线提示扫码，分别报告采集/AI 执行/投递可用；默认不清 runtime、重装、升级或重做结果不明任务。当前未实现/未验收，恢复入口优先级后移。

### B5. Gateway Runtime

- [ ] Poll Worker 从已有 Checkpoint/generation 继续，不重新 bootstrap。
- [ ] Dispatch Worker 由 Gateway Release/Compose 生命周期独立启动，heartbeat fresh，Hermes reachability 真实检查通过。
- [ ] Controller `start` 后 Poll Worker 与 Delivery Worker 均 healthy、heartbeat fresh，agent-wechat auth/API 可用。
- [ ] Runtime health 无未归属的 `uncertain`、stale lease、blocked thread、missing delivery 或 poison candidate。
- [ ] Queue backlog 和 oldest age 在批准阈值内。

### B6. 文本链

- [ ] 获准私聊只形成一个 Message、Admission、Dispatch、Response、Delivery 和可见回复。
- [ ] 未获准私聊持久化但不调用 Hermes。
- [ ] 先检查员工 Bot 使用权限，再检查其群聊使用权限；真实 `@` 只由获准员工发起，其他参与者作为任务上下文不自动获权。
- [ ] 群聊未 `@` 为 `bot_not_mentioned`，不调用 AI。
- [ ] 纯文本伪 `@`、引用或上一条 mention 不被继承。
- [ ] self 发言防回环与消息留档分开：不触发 AI，同时核验实际归档覆盖；现存 Poll 跳过 self 与独立 Response/Delivery 记录不能证明所有 Bot 自发消息已留档。
- [ ] 重复来源事件只命中同一逻辑链。
- [ ] Bot 在线期间持续采集获准范围私聊/群聊消息；Poll 停止、范围/上游窗口及连续性缺口单列，不由“在线”推断全量。

### B7. Thread 与 Context

- [ ] 按 B 方案每员工、每群各有独立的真实 Hermes Profile 与持续会话；分组仅提供模板，同组不共享 Profile 状态，首次后续消息接续而非逐条新建。
- [ ] 员工私聊按员工分组、群聊按群分组选模板；已授权未分组不暗配默认 Profile。换分组不自动提升文件权限。
- [ ] 现存 V2 `group_sender`、V1 compatibility 与 `group_shared` 的实现/启用状态单列；既有按发送者隔离测试不代替最新每群持续会话验收，未经实现验证不提前切换。
- [ ] Profile 管理分别记录引用/版本保存、请求实际路由、自动创建、修改同步、已有会话生效五项；官方支持或请求携带 Profile 字段不等于全部接通。
- [ ] 模板同步只更新管理员明确管理字段，不覆盖聊天历史、已产生记忆、运行状态或独立凭据；字段范围、调组/重置与会话生效时机待细化/验证。
- [ ] 当前私聊/群聊执行中的补充实际进入对应运行过程；同线程 FIFO 和已结束后续接不作为通过证据，独立新请求及不可撤销动作后的修改分别记录。
- [ ] Context read/recent/range/search 限定获准身份/会话；当前群近几条消息、明确引用可用性与高级跨会话检索分开。
- [ ] Snapshot version 与 coverage 正确，未跨 unfinished turn。
- [ ] 引用正文只有在实际注入并验证后才能标记 PASS。
- [ ] 人从 Web 与机器人通过受控工具读取同一 Gateway 档案，按会话/人员/时间授权；不发管理员 Token 或数据库权限给模型。
- [ ] 区分 `occurred_at`/`timestamp`、`received_at`、`created_at`；日期查询主要按 `occurred_at` 起止区间，按北京时间某天/月换成明确区间并分页读完实际已归档且获准记录，不重新轮询历史，不一次塞入全部模型上下文。
- [ ] Web 分别核验已有数据库/读取 API、管理写 API、所需数据采集与页面；分组到电脑、运行/排队、资源与频率展示按真实覆盖记录，不写“只差页面”。调整电脑映射不默认丢历史、重复执行或无声切换，生效细节待细化。

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

依据[当前文件触发与历史业务证据](../requirements/file-workflows.md)，六项独立记录期望、实际、状态和脱敏证据。普通文件没有任务时一般不处理/不回复，不自动下载永久归档；按任务取得必要原件、明确要求才正式保存，纯文档群例外需显式配置。下表为历史可核对边界，不能直接勾选为未来 Release 的 PASS。

| 独立验收项 | 通过标准 | 现有结果与限制 |
| --- | --- | --- |
| 微信入站取得字节 | 获准消息实际取得可读取文件/图片，受控持久化与附件引用可核对；企业原始资料、ZIP 原件及成员核验完整性 | PDF 实际正文/附件/摘要一致；PNG 按当前业务口径已验收。ZIP 入站仍未完整验收；历史超时及一次失败说明投递保留，不判为损坏或格式不支持 |
| 内容理解 | 真实模型/适用工具实际读取内容，描述与样本一致；看不清/缺失信息待确认，不猜造商品归属、报告或授权事实 | PDF 正文与 PNG 实际读取、准确描述通过；后续文字同会话接续；不推广为所有格式、多轮/多人或复杂续办通过 |
| 按要求保存/分类归档 | 员工明确要求保存或命中已配置纯文档群规则后，在获准范围通过 FileBrowser API 保存；核对字节、来源和审计，未知/冲突询问，同名不同内容不静默覆盖 | 一个商品 4 主图、8 详情成功，原包保留；首次导入失败与后续新操作成功分列。非全企业资料库；目录/分类规则由后续 Hermes/Skills 承接，群权限与文件通道仍属平台；离线自动补分类专项撤出，路径调整明确暂缓 |
| 文件中心查询回传 | 按身份/会话权限查询唯一 File Service，实际选件、导出、READY Artifact/Delivery 可关联；不强制原件绕行 AI 主机 | 正常微信商品 ZIP 已由正式模型查询/导出新包并交付；受限 API 查询/复制/移动/改名/替换/可恢复删除/还原有证据，不等于每项均由真实模型选择 |
| 原聊天实际附件实收 | 原请求会话收到且可打开正确文件/图片；企业原始资料与 ZIP 核对自身导出/Artifact/收件整包和成员，确认不错投/不重复 | PDF、PNG 微信图片和正常 ZIP 出站实收已验证。**PNG图片识别及微信图片收发已验收，满足当前业务要求。** 原 PNG 字节差异、重编码和单独 file 附件路由不再列为待修阻塞；变化原因未定位。ZIP 18,167,063 字节、12 成员一致；文件名前缀/中文替换未通过原名验收，候选未生产验证 |
| 加工结果回存 | API 下载任务工作副本、加工/检查，再另存新文件或新版并回读、关联原件及审计，默认保留原件 | 客户端和素材归档/导出可复用；通用 Agent 加工回存、原件关联和完整版本管理仍需独立业务验收 |

- [ ] 六项分别写 PASS/FAIL/BLOCKED/NOT_RUN；成功文件操作不代替真实模型选用，单商品不代替全库，ZIP 出站不代替 ZIP 入站。
- [ ] 聊天消息/附件元数据留档、任务临时取得原件、正式文件库归档分别留证；D005 持久化和防重复保留，不默认第二天执行全部积压业务。
- [ ] PDF/PNG 原样回传只证明相应通道；“明确要求保存，再独立检索并交付”另留业务证据，不能相互替代。
- [ ] 合法原文件名保留，无无意义前缀或中文替换；已有候选/隔离验证与实际协议部署、微信收件验收分列，后两项仍缺，必须作为平台基础问题处理。
- [ ] READY Artifact、Delivery 与收件分别核对；不因投递不确定盲目重发，不重放成功操作补证。
- [ ] FileBrowser migration、backup/restore、rollback、WebDAV/OnlyOffice、权限/服务端审计关联逐项验收。
- [ ] 后续 Skills 由用户在四仓基础完成后找 Hermes 制作并单独建仓；当前不安排并行试点、不建新 Runtime/RAG，不因工具安装标记业务 Skill 已交付。
- [ ] 旺店通/S6 未接入部分保持 BLOCKED；不以文件链通过替代 ERP 业务。

最终部署标准优先级后移，后续另外验收，不用工具发现或既有兼容 CI 代替：

- [ ] 服务端一条正式可复用入口，必要参数完整，新环境可独立部署/恢复，组件 ownership 清楚。
- [ ] 电脑仅官方 Hermes 加少量配置，无额外项目 EXE、手工复制插件、源码修改或项目专用常驻进程。
- [ ] 不依赖 Codex 工作树、`.task-artifacts`、旧现场回执；正式业务凭据与临时运维权限分离。
- [ ] 临时运维权限撤销后，业务凭正式凭据实际继续运行并留证；撤销成功不等于此项通过。
- [ ] MCP 如被采用，分别验证发现、权限/撤销、真实文件工具、Artifact/Delivery 和完整业务；隔离发现成功不能宣布生产迁移。

最后现场记录中的插件/客户端桥接状态与最终目标分列。PR23 最终 `57bb2c2830f292d5167f0bda449afafbe0a528fd` 已验证官方完整依赖与原生图文/文件工具，但可信任务授权及返回链未过；PR24 最终 `b3f2740289c6b1fa3208c352bc87b99b6f50cce4` 已通过统一入口隔离容器检查，三组件 booted-systemd/持久宿主盘/整机重启仍缺。两者都不证明新的 Profile 管理或现场部署。完整条件见[部署指南](../deployment/deployment-guide.md#最终部署标准与候选边界)。

群发与高级跨会话完整档案智能检索继续后置；每群独立 Profile/持续群会话、当前群必要上下文、执行中补充和当前范围/日期档案查询是平台基础需求，不能一并拖到最后。Gateway Web 页面后续开发，本轮只记已确认目标、现有数据/API 与实际缺口。

### B12. 结果

- [ ] 记录 PASS、FAIL、BLOCKED、NOT_RUN 数量和原因。
- [ ] 记录消息丢失、重复、错投、权限和业务副作用判断。
- [ ] 记录 rollback 是否执行。
- [ ] 记录残余风险、owner 和复核时间。
- [ ] 最终决定为 GO、LIMITED-GO 或 NO-GO，并有签核。
