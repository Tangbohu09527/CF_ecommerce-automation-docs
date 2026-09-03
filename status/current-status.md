# 当前状态矩阵

> 状态日期：2026-09-03
>
> 本文是系统当前状态的唯一权威入口。每行分别说明仓库实现、部署和生产验证，任何一层通过都不能自动替代另一层。

## 状态口径

- **Repository implemented：** 能力存在于指定代码线。
- **Automated tests / GitHub Actions：** 只证明对应提交和测试环境。
- **Deployed：** 对应代码或组件已进入 CFserver/AI 主机。
- **Production validated：** 指定行为已在真实链路留证。
- **Planned / Not implemented / Not verified：** 不得写成已可用。

## 能力矩阵

| 组件或能力 | Repository implementation | Deployment | Production validation | Remaining boundary | Next action |
| --- | --- | --- | --- | --- | --- |
| `CF_agent-gateway` | `main`=`b488cf452584e73bc9b752564bf90ea153aa8d18`；V2 Runtime、revision `20260823_04`、P1 已合并，main CI 通过 | immutable image `sha256:b9341ca7df6f952b4d81028c497574c1e22478e4408f98791a28bd9514b215f1` 在线 | Gateway、Poll/Dispatch/Delivery、Controller、队列清空与文本链路通过 | PostgreSQL restore、全部故障场景和业务扩展未完成 | 保持 Release 证据并完成 restore/容量演练 |
| Gateway P1 observability | P1 PR #7 已合并到 Gateway main；结构化日志与降噪测试通过 | Release `p1-observability-main-b488cf452584-20260903`；Gateway 日志 `64m x 10` | 启动期有限摘要、稳态无重复目标日志、0 ERROR/violation 已验证 | 长期容量与集中告警仍属运维责任 | 建立持续容量监控和告警 |
| `CF_agent-wechat` | `main`=`92393bc2ae1d89dae9449fc131413979aa2fa2f2`；forced-QR 实现尚在 PR 栈 | 生产容器在线时使用 forced-QR R2 行为 | 登录、auth/chats/messages、文本收发通过 | 实现尚未 main promotion；现场 Image ID 与 exact source SHA 映射未证明 | 按 PR #4 -> PR #1 顺序复核提升 |
| forced-QR R2 | PR #1=`9cb7163…`，PR #4=`dc103c8…`，均 OPEN；PR #5 是未合并文档工作；PR #4/#5 CI 当前部分失败 | 当前生产契约为 `restart: "no"`、loopback 6174、`cf-internal`、`ENABLE_VNC=0`、Token 只读挂载 | fresh QR、手机扫码、API 放行和 Host reboot 后恢复已验证 | automatic boot stop gate 未完成；Archive 不得复用为 active Session | 修复或解释 CI 后完成正确 promotion |
| PostgreSQL | Gateway schema 与权威状态模型已实现 | CFserver healthy，revision `20260823_04` | Message/Admission/Dispatch/Response/Delivery 与队列一致性通过 | 实际 restore 演练未完成 | 在隔离恢复目标完成 restore 验收 |
| Hermes | 外部 Runtime，不属于本仓库或 Gateway 实现 | Windows AI 主机可达 | 私聊/群聊文本实际调用成功；AI 主机重启后 reachability 恢复一次 | watchdog、自启正式文档、告警、容量和高可用未完整验收 | 收口守护、启动和监控证据 |
| private text | Gateway/WeChat 文本链实现与测试存在 | 已部署 | 完整真实回复闭环通过 | 不代表媒体、Skills 或业务自动化 | 保持每次 Release 回归 |
| mentioned group text | 结构化 mention、Admission 和 V2 Routing 已实现 | 已部署 | 真正 `@` 可回复；未 `@` 不调用 AI | 不证明同群多发送者隔离已生产验收 | 增加双发送者生产对照测试 |
| thread isolation | V2 `group_sender` key 包含 sender identity，自动化测试覆盖；V1 compatibility path 仍是 whole-room | 生产 Runtime 使用 V2 代码线 | 私聊/群聊单路径隔离有证据 | 同群不同发送者生产对照未验证；`group_shared` 未批准 | 留证验证双发送者并继续禁用 `group_shared` |
| Context Runtime | Timeline、授权读取、Snapshot、search 与 Hermes context tool 已实现并通过 Gateway CI | 随 Gateway Release 部署 | 文本链使用线程绑定；Context 全能力未逐项生产演练 | RAG、Memory、引用正文注入未完成 | 验证 Snapshot/read/search 的生产行为 |
| Admin recovery | `uncertain` inspect、retry-approved、mark-dead、confirm-success 与 immutable audit 已实现并测试 | 随 Gateway Release 部署 | 已有一次证据核对与 Guard 的受控恢复 | 未证明每个动作都在生产演练；不得手工改 DB | 为每种动作建立脱敏演练记录 |
| Checkpoint continuity/rebase | generation、anchor、CAS rebase、历史前缀/实时后缀逻辑已实现和测试 | 已部署 | forced-QR 后回退、前缀跳过、后缀单次处理、self skip、无重复回复通过 | 长期上游保留窗口与更多异常分支需持续观察 | 纳入每次 forced-QR 回归 |
| Response / Delivery | durable Response、Outbox、Attempt、reconciliation 已实现 | Dispatch/Delivery Worker 已部署 | 文本持久化、投递、无重复和队列清空通过 | 完整媒体投递未完成 | 保持文本回归并接入 Artifact 链 |
| media discovery | 图片来源识别、Raw Payload 和读取接口已有实现 | 来源链已部署 | JPEG 字节、签名、大小与摘要验证已完成 | 只证明可发现/读取，不证明 AI 看图 | 接入 Attachment 和受控媒体存储 |
| full media pipeline | Gateway 仓库包含部分 Artifact/媒体数据结构与测试 | 系统级链路未完整接入 | Not production validated | 入站持久化、Hermes 多模态、出站物化和微信回传未完成 | 分阶段完成入站、推理、出站验收 |
| CFserver reboot | 核心服务 restart policy 与 WeChat `restart:no` 已配置 | 已执行一次真实重启 | Docker/存储/Gateway 核心恢复；agent-wechat 保持停止；fresh QR 后恢复在线 | Poll/Delivery 当时曾自动 running/healthy，automatic boot stop gate 未验证 | 启动后先检查并显式 stop Gate，再 fresh QR |
| AI host reboot | Hermes 外部运行能力存在 | 已执行一次真实 AI 主机重启 | Hermes reachability 恢复，WeChat Session 保持，无需 fresh QR | 不等于完整 watchdog/告警/高可用交付 | 建立可重复自启和告警验收 |
| Gateway-only deployment | 不可变镜像切换与 Controller 已实现 | 多次受控 Gateway 切换 | agent-wechat 未重建，authenticated Session preserved | 不可外推到 agent-wechat 自身重启 | 固化为独立发布场景 |
| FileBrowser | `main`=`4750a97…`；`feat/v1-integration`=`48380c3…`，V1 Beta 核心实现与自动化验证完成，CI 对应代码线通过 | 尚未在 CFserver 正式部署 | Not production validated | 迁移、备份恢复、回滚、真实 WebDAV/OnlyOffice、Agent 集成未完成 | 完成 CFserver Candidate、部署与恢复验收 |
| Skills | Not integrated | Not deployed | Not verified | 权限、执行、幂等、审计和业务 Skill 均未接入 | 先定义 Skills Runtime 契约 |
| OCR | 第一阶段明确不建设独立 OCR | Not deployed | Not applicable to delivered scope | 未来是否需要取决于真实业务 | 不提前立项 |
| 旺店通 / S6 | Not integrated | Not deployed | Not verified | 接口、权限、确认、幂等和回滚未完成 | Skills 与文件边界稳定后分业务接入 |

## 当前结论

企业消息与 AI 文本闭环基础已经生产交付。六阶段规划中的阶段 1 仍含文件基础链路，因此当前应表述为“阶段 1 的消息 Runtime 里程碑完成，文件基础链路继续推进”，而不是把整个阶段或整个项目标记为完成。

动态 Message、Checkpoint、Queue、容器和 Archive 数量不在本文维护；带日期的实际结果见[2026-09-03 Production Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。
