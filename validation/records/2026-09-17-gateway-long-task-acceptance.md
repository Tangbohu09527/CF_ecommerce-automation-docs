# 2026-09-17 Gateway 长任务回传验收摘要

> 结果：PASS，仅限本次分钟级任务的 Gateway 等待、响应持久化及微信回传。
>
> 来源：用户在本轮协作会话提供的 CFserver 输出与微信截图；本次文档整理未远程连接生产。
>
> 本记录增补最新 Gateway 证据，不覆盖[2026-09-03 总体生产基线](./2026-09-03-enterprise-runtime-production-closeout.md)，也不将所有其他组件重新标记为已验收。

## 组件权威与文档引用

| 对象 | 带日期事实 |
| --- | --- |
| Gateway branch authority | `main` |
| 2026-09-17 核对的 main / PR #11 merge | `9a1caa237a9053678c80f68fdb15d351d5bfecf8` |
| 现场 Docker Image ID | `sha256:1cd7650543babe75d4fabe71e27e3cbc1d54585d34ffa853280606c2a3ddaa8b` |
| Gateway 文档提交 | `6a430eab6a6ef752c7e88583732f299162cdca59`，文档分支 `docs/long-task-acceptance-20260917` |
| Gateway 文档 PR | [PR #12](https://github.com/Tangbohu09527/CF_agent-gateway/pull/12)，本记录编写时 OPEN，未合并 |
| 企业总文档起点 | main 的 dated snapshot `8f51cd095c6967f806701b703281b08e5144296f`；在独立文档分支补充本记录 |
| Schema | Gateway 仓库 head `20260823_04`；现场 database/migration_schema 为 ok，文档整理未再次查询现场 revision |

完整明细归属组件仓库：[固定提交的 Gateway 验收记录](https://github.com/Tangbohu09527/CF_agent-gateway/blob/6a430eab6a6ef752c7e88583732f299162cdca59/docs/validation/2026-09-17-hermes-long-task-acceptance.md)。
引用固定文档提交使 PR 尚未合并时也能核对证据，不表示该文档提交已经是组件 main。代码 PR #11 已合并，与文档 PR #12 是否合并是两件事。

Image ID 不是经核实的 registry manifest digest。代码合并和观察到的镜像分别核验，完整构建来源映射、新 Release label/Tag、新离线归档及恢复/回滚材料未独立证明。9 月 3 日 P1 旧镜像与旧归档不替代这些新证据。

## 实现、部署、测试与现场验证分层

| 层次 | 结论 |
| --- | --- |
| Repository implementation | PR #11 有限等待及不确定派发保护已合并 |
| Automated tests | PR #11 原记录的 444 项相关回归属于其原环境，不是本次文档的新测试 |
| GitHub Actions | 本次文档 PR 按各自提交另行查 CI，不借用旧绿灯 |
| Deployment | 用户的现场输出核对四个应用 Image ID、Dispatch 600 秒启动配置、3660 秒停止宽限期与 Controller/Token 契约 |
| Production validation | 以下唯一测试消息的数据库证据与微信实收匹配 |
| Not reviewed / Not run | 工具原始日志、Desktop 构建、近上限/断线/重启/并发/FIFO/续租专项及完整归档恢复 |

## 本次预期与实际

测试编号 `CF-LONG-20260917-02`：微信请求外部 Windows Hermes 同步等待 90 秒，再只读既有合成测试文件，等待执行结束后返回。本文不发布 Windows 路径、文件名、消息正文、账号或截图。

| 证据 | 实际结果 |
| --- | --- |
| 入站唯一匹配 | Message `1634`，匹配数 `1` |
| Dispatch | `24`，success，尝试 `1`，错误码为空 |
| 服务端派发时间 | `05:34:29.365607Z` 至 `05:36:23.164729Z`（均为 2026-09-17） |
| 数据库派发时长 | **113.799 秒** |
| 响应 | 原始响应 `1` 条；标准化 Response delivered，`1` 段 |
| Delivery | `23`，delivered，尝试 `1`，完成时间 `05:36:26.489926Z` |
| DeliveryAttempt / Receipt | Attempt `23`，分段 `0`、第 `1` 次尝试，匹配回执 `1` 条 |
| 派发完成至投递完成 | **3.325 秒** |
| 回复核对 | 测试编号、运行编号、时间、路径、内容、哈希与本次私有验收预期及截图相符 |
| 最终数据库核验 | `LONG_TASK_DB_CHAIN=PASS` |
| 微信实收 | 用户截图中的结果与落库回复匹配 |

回复报告工具耗时为 90.001 秒；Gateway 的 113.799 秒由数据库起止时间独立计算，不是引用模型自行填写的耗时。该服务端时长仍不等于每个工具/HTTP 子阶段的精确测量。

原始 Dispatch 响应条数已核对；并未逐字比对原始响应正文与标准化分段。标准化回复的匹配及哈希自洽不能替代独立审阅 Hermes 工具日志。`RAW_HERMES_TOOL_LOG=NOT_REVIEWED` 明确保留。

## 恢复历史与残余问题

历史 Dispatch `21` / Message `1349` 在 2026-09-16 17:35:03.640393（北京时间）有 Recovery audit `1`，独立 `mark_dead` 记录 uncertain → dead；次数保持 1，不再阻塞。没有伪造成功或重派，保留错误和审计。

排队只读测试 Message `1403` → Dispatch `22` → Delivery `21` 也已回传，派发耗时 20.444 秒，各尝试一次。Delivery 21 不是历史 Dispatch 21。

首次长测试 `CF-LONG-20260917-01` 的回复报告脚本解析错误，未开始 90 秒等待，不能计作长任务通过。本次 `-02` 是新测试，不是改写旧失败结果。

已审阅 Poll 日志中三个历史会话连续性失败尚未关闭：两个 visible-window-empty，另一个 Checkpoint `18` 的 empty-window-marker-unavailable。验收会话 Checkpoint `12` 轮询成功且非 unverified，未通过手改 Checkpoint/generation/fingerprint 放行。

这些是带日期观察，不是全局持续状态。保留 dead 和历史连续性问题可能使总健康继续 degraded；不得删记录以变绿。

## 未完成范围与后续责任

| 范围 | 状态 / 后续 |
| --- | --- |
| 原始工具命令、标准输出、退出码和桌面版本 | NOT_REVIEWED；由现场运维/用户补充受保护证据 |
| 镜像完整构建来源、新归档、备份恢复/回滚 | NOT_VERIFIED；独立核验，不复用旧 P1 成功标签 |
| 近 600 秒、执行中断线/重启、并发/FIFO/续租 | NOT_RUN；需单独批准和留证 |
| 历史会话连续性恢复 | OPEN；逐项处理，不重置读取锚点绕过保护 |
| 完整自启/watchdog/HA、全部会话健康 | 未由本次证明 |
| FileBrowser/企业 Skills/媒体/ERP | 不属于本次通过范围，维持各自待办 |

本次文档保存脱敏摘要，不上传完整终端历史和微信截图；尚未提供可再次核验的受保护离线归档路径/校验和。外部 Hermes、数据库、主机与微信入口仍各有其现场责任边界。

## 操作边界

只修改企业总文档 Markdown，组件完整证据由 Gateway 文档 PR 承载；本次不修改生产代码、Compose、配置、Workflow 或数据库，不 SSH/Docker 操作现场，不创建 Tag、不自动合并任何 PR。

本次限定场景通过不等于阶段 1 文件链路或整个企业自动化系统完成。下一步在[当前进度](../../status/current-progress.md)跟踪。
