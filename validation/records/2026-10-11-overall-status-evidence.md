# 2026-10-11 总体状态脱敏证据摘要

> 结论：新候选与两端新 plan 已通过，尚未切换；用户随后已重启 CFserver，重启后状态未核验。微信真实附件业务尚未通过端到端验收。
>
> 现场来源：用户 2026-10-11 在本次任务提供的终端回传摘要与明确的 reboot 说明。本次未连接 CFserver/Windows、未读取生产凭据或业务文件，未执行生产命令或完整业务验收。
>
> GitHub 来源：2026-10-11 18:23（Asia/Shanghai）前完成四仓只读查询，记录固定提交、main、PR 与精确 SHA 的 Actions。此后 main/PR 变化需另查；不等待其他文档任务合并，也不假称引用其尚未完成成果。

## 1. 总文档起点与保留范围

- 仓库：`Tangbohu09527/CF_ecommerce-automation-docs`。
- 起始分支 `docs/file-business-workflows-20260927` 的远端 head 与本地干净起点均为 `8179533ffade1027d01727cbf5a89a26b491c400`。
- [原 PR #9](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/9) 为 OPEN/Draft，head 为上述分支/提交，base 为 main；本次保留它及[9 月 27 日文件业务需求](../../requirements/file-workflows.md)。
- 本次独立 worktree/分支 `codex/docs-overall-status-20261011` 的 Draft 文档 PR base 是 `docs/file-business-workflows-20260927`，不是 main。
- 核对时文档 main 为 `dd81075d7516d74a9e307ebdc03ce8ec07e12c49`。仓库无 Actions Workflow，固定起点的 runs/check-runs 均为空；不得写“文档 CI 通过”。本次检查与最终提交记录在新文档 PR。
- 原六阶段保持；当前集中真实微信文件/图片收发，不并行扩安装器、大重构或 ERP 集成。

## 2. 三组件代码、合并与 CI

以下链接固定到 commit/run；main 仅是此次查询快照，不是生产镜像来源证明。

| 组件 | 固定候选及分支 | main 快照 / PR 状态 |
| --- | --- | --- |
| Gateway | [`d9bae61a043f1be4303e5c23661ec51904d5e708`](https://github.com/Tangbohu09527/CF_agent-gateway/commit/d9bae61a043f1be4303e5c23661ec51904d5e708)，`codex/fix-hermes-env-loopback` | main=`7963bc0db5a01e38099e021959feabf648db7ede`；候选 ahead 43/behind 0，未合并、未发现关联 PR。[PR #14](https://github.com/Tangbohu09527/CF_agent-gateway/pull/14) 仍 OPEN/Draft，head `feat/wechat-inbound-media@f9c601cba25cbc5459bb03519d1954ae55995ded`，base main；d9 在 f9 之后再有两个修复提交 |
| WeChat | [`9b997ac92d653380a01ac93e0ac65e4a0e3a4c0a`](https://github.com/Tangbohu09527/CF_agent-wechat/commit/9b997ac92d653380a01ac93e0ac65e4a0e3a4c0a)，`codex/fix-wechat-original-media` | main=`81d21f451adc61df71077ca3035adeddeca4621f`（[PR #7](https://github.com/Tangbohu09527/CF_agent-wechat/pull/7) 已合并）；候选 ahead 2/behind 0，未合并、未发现关联 PR |
| FileBrowser | [`bb11be4944b5325f2d10aa5a1adb88a3f467da72`](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/commit/bb11be4944b5325f2d10aa5a1adb88a3f467da72)，`feat/filebridge-hermes-crud` | main=`4750a97cfdf5bd067e04b6b36bf9616f5ada836d`；[PR #2](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/2) OPEN/Draft，base `feat/file-service-clean-device@8ff37ef0da322aa2194d9ef37e26eb7f7035cbb2`，未合并；其前置 [PR #1](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/1) 亦 OPEN，目标 `feat/v1-integration` |

| 对应提交 | GitHub Actions 结果 | 隔离测试与限制 |
| --- | --- | --- |
| Gateway d9 | [CI 38127068913](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/38127068913)、[Hermes compatibility 38127068876](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/38127068876)、[Clean device 38127068875](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/38127068875) 均 success | CI 相关回归 2477 passed / 3 skipped；含 host、迁移/PostgreSQL/V2 检查。官方 Hermes 回环/HTTPS 附件与 Windows 管理重启、Debian daemon/QEMU 安装重启是隔离环境证据，不证明 CFserver 新候选已运行 |
| Gateway main 快照 | [CI 35192911684](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/35192911684)、[Clean device 35192911658](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/35192911658) success | 9 月 17 日该 SHA 的历史检查，不替代 d9 或 10 月 11 日现场 |
| WeChat 9b | [媒体隔离 36403240631](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/36403240631)、[forced QR 36403240630](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/36403240630)、[Debian clean host 36403240644](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/36403240644) 均 success | 9 check runs 成功；上游 Rust 55 passed，其中 6 项缓存契约使用合成 fixture，不证明登录态微信真实下载/发送/实收 |
| WeChat main 快照 | [forced QR 34020705968](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/34020705968)、[Debian clean host 34020705965](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/34020705965) success | 对应 9 月 6 日 main，非媒体候选生产验收 |
| FileBrowser bb11 | [FileBridge client 37407661714](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/37407661714)、[Windows config 37407661618](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/37407661618)、[deployment-assets 37407661634](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/37407661634) success；[常规测试 37407661615](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/37407661615) **failure** | 客户端四个作业已成功，PR 正文中“3/4 运行中”为旧文字；常规测试 lint-backend 失败，其余六个作业成功，不能写全仓 CI 绿 |
| FileBrowser main 快照 | [release 29586464929](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/29586464929) success | 7 月 17 日 main 的历史结果，不证明 bb11 已合并或生产发布 |

### 固定代码/文档能证明什么

- Gateway d9 修复正式 plan 的 Hermes `.env` HOST、truststore TLS 兼容、原 baseline 标签身份解析及 `complete-rollback`；候选镜像身份门禁仍保留。[回退续接记录](https://github.com/Tangbohu09527/CF_agent-gateway/blob/d9bae61a043f1be4303e5c23661ec51904d5e708/docs/validation/2026-10-11-server-rollback-resume.md)与[环境/回环记录](https://github.com/Tangbohu09527/CF_agent-gateway/blob/d9bae61a043f1be4303e5c23661ec51904d5e708/docs/validation/2026-10-11-hermes-env-loopback.md)中的本地/隔离通过，不替代本次现场事实。d9 仓库 schema head 为 `20260928_01`；旧现场记录 `20260823_04` 不可直接写作新候选目标或重启后实际 revision。
- Gateway [10 月 7 日文字取文件记录](https://github.com/Tangbohu09527/CF_agent-gateway/blob/d9bae61a043f1be4303e5c23661ec51904d5e708/docs/validation/2026-10-07-wechat-file-api-text-closeout.md)只证明微信文字 → File API → 副本读取 → 原聊天文字，且记录模型传输图片字节未观察到；不等于微信入站附件取得或原聊天附件实收。
- WeChat [媒体候选记录](https://github.com/Tangbohu09527/CF_agent-wechat/blob/9b997ac92d653380a01ac93e0ac65e4a0e3a4c0a/docs/validation/2026-09-28-media-retrieval-candidate.md)及[API 文档](https://github.com/Tangbohu09527/CF_agent-wechat/blob/9b997ac92d653380a01ac93e0ac65e4a0e3a4c0a/docs/api.md)说明原生下载请求和缓存长度/MD5检查。图片 best/full 是缓存层级，full 仍需原样本摘要/转码对照才能证明原件。该分支新增来源锁、隔离核验和文档，未修改生产 Compose/安装脚本；历史图片可读取不推导图片/文件可发送。微信上游锁定候选与下文 Gateway 新候选镜像是不同制品。
- FileBrowser bb11 的[10 月 6 日原生读取记录](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/blob/bb11be4944b5325f2d10aa5a1adb88a3f467da72/integrations/hermes-filebridge/validation/2026-10-06-native-media-read.md)记录：批准 scope 中既有 PDF/PNG 经真实 FileBrowser HTTPS 下载，原字节一致；官方 Hermes 本机 PDF 正文与真实辅助视觉独立比对 42/42。它不是微信入站、完整 Agent 会话或附件回传/加工回存验收。

## 3. 用户 10 月 11 日现场事实与截止点

下列按用户提供的先后顺序整理，不补造时间、主机 PID、Message/Checkpoint/Queue/容器/heartbeat/Archive 计数，也不复制长命令或完整终端日志。

| 顺序 | 现场事实 | 结论边界 |
| --- | --- | --- |
| 1 | 旧 f9 两端回退完成 | 已完成步骤，不再作为当前故障断点 |
| 2 | d9 控制器 `complete-rollback` 及旧核心恢复完成 | 不要求从头回退；不等于新 deployment 已切换 |
| 3 | d9 新候选镜像与两端新 plan 均通过 | 有候选与计划，但尚未 apply/start/verify |
| 4 | 重启前旧 Hermes、Gateway API、Dispatch 运行；Poll/Delivery 停止，Gate 关闭，数据库正常 | 仅重启前观察，不证明所有组件或重启后健康 |
| 5 | 用户明确已执行 `sudo reboot`，只重启 CFserver | 重启后状态未核验；Windows 是否关机未确认，不能推断 Windows 同时重启或仍在线 |

本次待接续 deployment：`cf-return-env-loopback-20261011-r1`。

用户回传的新候选镜像标识：`sha256:bcb3e42fef5ba5a91232ca04e49a959cbecab4b45eb3bec0a0b9b3aadf717185`。本任务没有独立核实其 registry manifest digest、完整构建来源或当前运行容器映射，不因 d9 CI 成功自动补齐这些证据。

新 deployment **尚未 apply/start/verify**；原聊天实际附件实收 **尚未验收**。本文不把现场旧核心恢复、新候选计划通过、CFserver reboot 和生产业务验收合成一个“已恢复/已上线”结论。

## 4. 已有文件证据与六项业务验收

| 验收项 | 当前证据 | 尚缺什么 |
| --- | --- | --- |
| 微信入站取得字节 | 历史 JPEG 提取；WeChat 9b 隔离缓存契约 | 当前批准 PDF/图片通过微信取得完整字节、来源与摘要对照、Gateway 受控持久化的现场闭环 |
| 内容理解 | 9 月 26 日受限文件读取；bb11 10 月 6 日 PDF/PNG 本机原生理解；10 月 7 日文字取文件链 | 与微信入站同一原件绑定的实际内容处理证据；不能由文件名、元数据或读取成功推出理解 |
| 指定目录/分类归档 | 9 月 27 日受控文本新建；分类规则为既有需求 | 授权群/身份/目录规则、真实文件分类与商品匹配、待确认/去重/正式 API 归档与审计 |
| 文件中心查询回传 | File API 查询读取与文字回复有局部证据 | 受限检索选中正式文件、READY Artifact/Delivery 与实际附件回传关联 |
| 原聊天实际附件实收 | 历史文本实收；当前新 deployment 未验证 | PDF 文件方式与图片消息分别在原聊天实收、可打开、摘要/转码符合预期且无重复错投 |
| 加工结果回存 | 客户端读写基础与文本新建 | 同一任务 API 下载工作副本、实际加工、API 另存新文件/新版、保留并关联原件、回读及审计 |

六项均不能因文本新建、读取成功、HTTP 200 或 CI 绿而互相代替。[9 月 17 日长任务](./2026-09-17-gateway-long-task-acceptance.md)成功继续保留原日期及限定范围，不代表附件成功或本次重启恢复。

## 5. 接续顺序与未核验项

1. 由后续现场任务先核验 CFserver 重启后的实际状态、Controller/Gate、数据库、旧核心与 Windows/Hermes 可达性；不沿用历史 PID、旧健康或计数。fresh QR 前必须显式检查并关闭组合 Poll/Delivery Gate，Archive 不复用。
2. 核对**现存**新 deployment、候选标识与两端 plan，以及 reboot 后目标/基线是否仍匹配；符合条件才从尚未执行的切换环节接续。没有新证据要求时，不重新回退、传包、构建或重放 apply。
3. 接续切换并完成其 verify 后，按[六项验收清单](../production-validation-checklist.md#b11-媒体文件和业务扩展)验证真实 PDF/图片。计划通过、HTTP 200 和文本回复都不是附件实收。
4. 文件/图片收发后继续原 9 月 27 日业务主线：指定资料群分类归档、工作副本加工回存。统一安装器、大重构、轻量 Gateway Web、旺店通/S6 均不进入本轮实施。

仍未知/未完成：重启后所有运行状态、Windows 电源状态、新 plan 当前有效性、新 deployment 切换与 verify、完整镜像来源/运行映射、六项端到端文件业务、FileBrowser 全仓 CI 红项收口及组件 PR 合并。完整备份 restore、WebDAV/OnlyOffice、同群多发送者、watchdog/HA 等仍按各自历史证据边界，不因本次同步扩大为通过。

本次只完成文档事实同步与文档检查；后续现场核验不是本次已执行动作。
