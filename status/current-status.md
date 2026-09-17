# 当前状态矩阵

> Gateway 证据更新：2026-09-17。其他组件的仓库/部署状态沿用 2026-09-04 记录，除下文明确列出的现场观察外，本次没有重新核验其最新状态。
>
> 本文是系统当前状态摘要的权威入口。原始证据和动态记录编号放在带日期验收记录，不将一次检查写成持续监控。

## 状态口径

- Repository implemented：能力存在于指定代码线。
- Automated tests / GitHub Actions：只证明对应提交和环境，旧绿灯不能作为新提交的 CI。
- Deployed：有现场制品/配置证据；源码 SHA、Image ID、registry digest 和完整来源证明不能混用。
- Production validated：指定行为有带日期的真实链路证据。
- Planned / Not implemented / Not verified：不得写成已可用；未合并文档 PR 不替代 main。

## Gateway 新证据与历史基线

| 层次 | 2026-09-17 已核对 | 边界 |
| --- | --- | --- |
| 仓库代码 | PR #11 已合并；核对时 main 为 `9a1caa237a9053678c80f68fdb15d351d5bfecf8` | 这是 dated snapshot，不是永远不变的 live tip |
| 现场制品 | 四应用 Docker Image ID `sha256:1cd7650543babe75d4fabe71e27e3cbc1d54585d34ffa853280606c2a3ddaa8b` | 未独立证明完整源码/制品映射，未核实 registry manifest digest、新 Release label/Tag 或离线归档 |
| 配置与运行 | Dispatch 启动 read/execution=600/600 秒，停止宽限期 3660 秒；Controller/Token 契约和数据库 schema 检查通过 | 不代表所有会话健康或无人值守恢复通过 |
| 实机限定验收 | 数据库记录的分钟级派发为 113.799 秒；响应、单次投递尝试、回执与微信实收匹配 | 原始工具日志未独立审阅；600 秒上限、执行中断线/重启、并发/FIFO/续租专项未完成 |
| 文档同步 | Gateway PR #12 在本记录时 OPEN，证据文档固定提交为 `6a430eab6a6ef752c7e88583732f299162cdca59` | 引用该提交不表示文档已合并 main，本次文档工作不部署 |

9 月 3 日的 Production Release authority `b488cf452584e73bc9b752564bf90ea153aa8d18`、
source snapshot `f36c798294368263433f6132366ac9a864d9482b` 和旧镜像
`sha256:b9341ca7df6f952b4d81028c497574c1e22478e4408f98791a28bd9514b215f1`
保留为历史，不再描述成新现场镜像。9 月 4 日的 PR #8/#9 docs-only closeout 和旧 CI 成功仍是历史事实，不意味着后续没有发生应用升级。

详细证据见[2026-09-17 长任务回传](../validation/records/2026-09-17-gateway-long-task-acceptance.md)。

## 能力矩阵

| 组件或能力 | 实现/部署证据 | 生产验证 | 剩余边界与下一步 |
| --- | --- | --- | --- |
| Gateway V2 / P1 | 既有实现/测试及历史生产交付；本轮新镜像/配置如上 | 历史文本链路与本轮分钟级持久化回传通过 | 完整发布来源、离线归档及恢复专项待核验 |
| Gateway P1 observability | 9 月 3 日 P1 Release 与 `64m x 10` 策略历史记录 | 当时启动有限摘要、稳态降噪验收通过 | 本次未重复完整日志容量/保留周期验收；长期集中告警待办 |
| `CF_agent-wechat` | 沿用 9 月 4 日 main snapshot `69f07702b6ee16d8e9700b3a53d5ebbb8ee875f8`；PR #1/#4/#5/#6 合并和 CI Run `33863104399` 为历史记录 | 本轮现场登录、Gate 恢复和文本投递有证据；forced-QR reboot 仍按 9 月 3 日记录 | 本次未核对其 live main 或 source/image exact mapping；automatic boot stop gate 未完成 |
| forced-QR R2 | 历史 promotion/文档完成；契约为 `restart: no`、受限网络与 Token File | 9 月 3 日 forced-QR 行为验收 | 本次未重建微信入口或重新执行 CFserver reboot；不能新增无人值守恢复结论 |
| PostgreSQL | 仓库 head `20260823_04`；本轮 runtime database/migration_schema 为 ok | 消息/派发/响应/投递只读一致性证据 | 实际 restore 演练和新版备份材料未独立核验 |
| Hermes | 外部 Windows runtime，不归 Gateway 实现 | 本轮任务结果经 Gateway 回到微信；AI 主机重启后通路恢复有观察 | 原始工具日志/桌面构建、watchdog、告警、容量、高可用待核验 |
| private text | 已实现、部署；当前验收会话轮询通过 | 历史文本闭环及本轮长任务微信实收 | 不代表媒体、企业文件服务或 Skills 集成 |
| mentioned group text | 结构化 mention、Admission 和 V2 Routing 已实现/部署 | 历史真正 @ 回复、未 @ 不调用 AI | 本轮未新增群聊对照验收 |
| thread isolation | V2 `group_sender` 按 sender identity 隔离，自动化覆盖；V1 compatibility 保留 whole-room | 历史私聊/群聊单路径证据 | 同群双发送者及长任务跨会话/FIFO 专项未完成；`group_shared` 未批准 |
| Context Runtime | Timeline、授权读取、Snapshot/search/context tool 已实现并随 Release 部署 | 文本线程绑定有证据 | 全能力生产演练、RAG/Memory/引用正文注入未完成 |
| Admin recovery | uncertain inspect/retry-approved/mark-dead/confirm-success 与不可变审计已实现/测试 | 本轮补充独立 mark_dead 的生产审计证据 | 不能把单个动作外推为所有恢复动作已演练；禁止手改 DB |
| Checkpoint continuity/rebase | generation/anchor/CAS 及前缀/后缀逻辑已实现/测试 | 历史 forced-QR 回退、单次处理有证据；本轮验收会话未受影响 | 独立历史会话仍有连续性保护告警，不重置 Checkpoint 变绿，单独处置 |
| Response / Delivery | 持久响应、Outbox、Attempt、receipt、reconciliation 已实现/部署 | 本轮长任务数据库与微信实收证据通过 | 不证明完整媒体投递或所有断线恢复 |
| media discovery | 历史来源识别、Raw Payload/读取接口 | 曾验证 JPEG 字节、签名、大小与摘要 | 不等于 AI 看图；受控存储/Attachment 链仍待办 |
| full media pipeline | 部分 Artifact/媒体结构和测试存在 | 未完成系统级生产验收 | 入站持久化、Hermes 多模态、出站物化与微信闭环待办 |
| CFserver reboot | 历史 Docker restart policy 与 WeChat restart:no | 9 月 3 日核心恢复和 fresh QR 记录 | 本轮未重新执行；automatic boot stop gate 未验证，fresh QR 前仍须正式关 Gate |
| AI host reboot | 外部运行能力与历史 reachability 记录 | 本轮 AI 主机重启后通路恢复、未因该次重启重新扫码 | 不等于执行中重启、完整自启/watchdog/HA 验收 |
| Gateway-only deployment | 不可变制品切换与 Controller 已实现 | 本轮恢复保持外部微信入口/数据库不重建，有分步骤核对 | 不外推到微信入口自身重启或全自动升级 |
| FileBrowser | 沿用 9 月 4 日记录：main=`4750a97…`、feat/v1-integration=`48380c3…`，V1 Beta 实现/自动化验证完成 | 当时 CFserver 部署/生产验收待办；本次未重新核查进度 | 迁移、备份恢复、回滚、WebDAV/OnlyOffice 和 Agent 集成待独立接管 |
| Skills | 既有状态未集成/未部署 | 本次不新增此类验收 | 外部 Hermes 本地文件测试不等于企业 Skills Runtime |
| OCR | 第一阶段不建设独立 OCR | 不属于已交付范围 | 不提前立项 |
| 旺店通 / S6 | 既有状态未集成/未部署 | 本次不新增此类验收 | 文件与 Skills 边界稳定后分业务接入 |

## 当前结论

阶段 1 的消息 Runtime 里程碑已有交付，2026-09-17 增补分钟级任务回传场景通过；阶段 1 文件基础链路和整个自动化系统仍未完成。

动态 Message/Dispatch/Checkpoint/Queue/容器编号和计数只放入带日期记录，不在此保存成永久现状。
历史基线见[2026-09-03 Production Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)，最新 Gateway 证据见[2026-09-17 记录](../validation/records/2026-09-17-gateway-long-task-acceptance.md)。
