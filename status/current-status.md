# 当前状态矩阵

> 同步日期：2026-10-11。GitHub 只读截止与用户现场先后顺序、完整 SHA/PR/run 链接统一见[证据摘要](../validation/records/2026-10-11-overall-status-evidence.md)。本次未连接生产；重启后状态未知，不给完成百分比。

## 状态口径

代码存在、合并、CI/隔离测试、实机操作、业务验收、规划与未知分别记录。main 是查询时快照；未合并分支不是 main。源码 SHA、Image ID、registry digest、构建来源与真实接收不能互相推导。动态消息、队列、容器、heartbeat、Archive 计数只进入带日期记录。

## 四项目矩阵

| 项目 | 代码 / 合并状态 | CI / 隔离测试 | 实机操作证据 | 业务验收 | 规划 / 未知 |
| --- | --- | --- | --- | --- | --- |
| 总文档 | 固定起点 `8179533…`；原 PR #9 OPEN/Draft；main=`dd81075…`。本次独立分支的 PR base 为 `docs/file-business-workflows-20260927` | 仓库无 Workflow；固定起点无 runs/check-runs。本次只做文档检查，结果见本次 PR | 不执行生产动作 | 不新增业务通过结论；保留六阶段及 9 月 27 日需求 | 本次 Draft 文档 PR 待审查，原 PR #9 保留、不合并 |
| Gateway | d9=`d9bae61…`，`codex/fix-hermes-env-loopback` 未合并、无关联 PR；main=`7963bc0…`。原 PR #14 OPEN/Draft，head 仍 f9 | d9 CI、官方 Hermes compatibility、clean-device 均 success；隔离 TLS/回环/重启通过；仓库 schema head `20260928_01` | 用户 10 月 11 日：旧 f9 两端回退、d9 complete-rollback、旧核心恢复完成；新候选及两端新 plan 通过，未切换 | 9 月 17 日分钟级文本回传；10 月 7 日微信文字→File API→副本读取→原聊天文字有固定记录；附件实收未验收 | 新 deployment 未 apply/start/verify；CFserver reboot 后核心/Gate/数据库、现存计划有效性未知 |
| WeChat | 9b=`9b997ac…`，`codex/fix-wechat-original-media` 未合并、无关联 PR；main=`81d21f4…`（PR #7 merged） | 候选媒体/forced-QR/Debian 三 run success；缓存契约为合成样本，不能推出真实原生下载或发送成功 | forced-QR 与文本收发保留历史日期；本次 CFserver reboot 后入口状态未核验 | 历史图片字节提取及文本实收；当前 PDF/图片真实入站与原聊天附件实收未形成完整验收 | 微信候选与 Gateway 新候选是不同制品；发送、原件获取、fresh QR/Gate 与当前镜像映射待现场核验 |
| FileBrowser | bb11=`bb11be4…`，`feat/filebridge-hermes-crud`；PR #2 OPEN/Draft，base 为 PR #1 的功能线，均未合并；main=`4750a97…` | bb11 客户端四作业、Windows config、deployment-assets 成功；常规 CI 的 lint-backend **失败**，不写全仓绿 | 9 月 25—27 日同机部署、HTTPS/受限账号接入与限定读写；10 月 6 日真实服务既有 PDF/PNG 下载 | 9 月 26 日文件读取；9 月 27 日受控文本新建；10 月 6 日官方 Hermes 本机 PDF/图片理解有证据；非微信附件闭环 | reboot 后服务状态未核验；完整群分类归档、原附件回传、加工回存、服务端跨层审计与正式交付门禁待完成 |

精确提交与 Actions 链接集中于[10 月 11 日摘要](../validation/records/2026-10-11-overall-status-evidence.md#2-三组件代码合并与-ci)。FileBrowser “从未部署/从未接入”的旧结论已被限定现场证据取代，但完整生产交付仍未通过。其他组件文档任务未完成时，本矩阵按上述固定证据截止，不引用尚未完成成果。

## 当前运行断点

旧 f9 两端回退、d9 控制器 complete-rollback 和旧核心恢复均已完成。随后新候选镜像及两端 plan 通过，尚未执行新 deployment 的 apply/start/verify。

用户随后明确只重启 CFserver。重启前旧 Hermes、Gateway API、Dispatch 运行，Poll/Delivery 停止、Gate 关闭、数据库正常，**仅是重启前事实**。重启后状态、Windows 是否关机以及计划在重启后是否仍适用均未核验；不沿用历史 PID。

下一步由后续现场任务先核验重启后状态与现存新计划，符合条件再接续切换。fresh QR 前显式检查并关闭组合 Poll/Delivery Gate，保持 forced fresh QR、`restart: no` 与 Archive 不复用。当前不从头回退、传包或构建。

## 文件业务分项

| 验收项 | 已有证据及边界 | 当前缺口 |
| --- | --- | --- |
| 微信入站取得字节 | 历史 JPEG 提取和 9b 隔离缓存测试 | 当前 PDF/图片完整字节、来源/摘要与 Gateway 持久化现场验证 |
| 内容理解 | 真实文件读取；bb11 本机 PDF/PNG 原生理解；10 月 7 日文字取文件 | 同一微信入站原件的实际内容理解，不以元数据/文件名/读取成功代替 |
| 指定目录/分类归档 | 受控文本新建只证明局部写入 | 授权规则、分类/商品匹配、待确认、去重、正式 API 入库及审计 |
| 文件中心查询回传 | File API 查询读取和文字回复有局部闭环 | 检索选件、授权、READY Artifact 与附件回传关联 |
| 原聊天实际附件实收 | 历史文本实收不覆盖附件 | 文件方式与图片消息分别实收、可打开、完整性及无重复错投 |
| 加工结果回存 | 客户端读写基础 | 工作副本加工、API 另存新文件/新版、保留/关联原件与回读审计 |

六项完整业务验收尚未完成。文本新建、读取成功、HTTP 200、CI 绿不能互相替代。详见[文件业务需求](../requirements/file-workflows.md)与[验收清单](../validation/production-validation-checklist.md#b11-媒体文件和业务扩展)。

## 历史已交付能力与持续边界

| 能力 | 有限证据 | 未完成/未知 |
| --- | --- | --- |
| 文本消息 Runtime | 私聊、真正结构化 @ 群聊、未 @ 安全结束、self 防回环；9 月 17 日长任务持久化和微信实收 | 不代表此刻在线、全部会话或全类型媒体成功 |
| Gateway V2 / Context / Admin | sender 隔离、Timeline/Snapshot/search、uncertain 管理有实现及自动化；9 月 17 日有 mark_dead 审计 | 同群多发送者、全 Context API、全部恢复动作独立生产验收 |
| Checkpoint continuity | generation/rebase、历史前缀跳过、实时后缀单次处理有历史验收 | 独立历史连续性告警仍按原范围跟踪，不重置 Checkpoint 清告警 |
| PostgreSQL | 旧生产记录 revision `20260823_04`；10 月 11 日重启前数据库正常 | 新候选 schema head 与现场实际 revision 分开；重启后状态、真实 restore 未核验 |
| Hermes / 恢复 | 官方 Hermes 外部执行与历史文本、一次 AI host reachability 恢复 | 当前 Windows 电源/运行状态未知；watchdog、执行中重启、近上限、并发/FIFO/续租、HA 未整体收口 |
| File Service 运维 | 唯一正式文件 API 与限定接入证据 | 全量迁移、备份 restore、回滚、WebDAV/OnlyOffice 与服务端 Persistent Audit 跨层关联仍需分项证据 |
| 发布 | 历史不可变制品与回滚记录；本次候选计划通过 | 新候选完整 provenance、registry digest、运行映射及业务验证不因 CI 通过自动成立 |

历史文本依据：[2026-09-17 长任务回传](../validation/records/2026-09-17-gateway-long-task-acceptance.md)、[2026-09-03 总体基线](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。历史 Image ID 与 Release 只在其带日期记录中解释，不能当作本次新部署。

## 规划范围

当前先交付微信真实文件/图片收发，再按 9 月 27 日既有需求推进内容分类、指定目录归档、查询分发与工作副本加工后 API 回存。AI 和员工共用 FileBrowser；AI 主机离线由 CFserver 先收原件、之后分类仍待实现/验收。

轻量 Gateway Web 仅待规划：局域网 IP 访问、Gateway 权限管理、查询 Gateway 已经接收的部分聊天/任务记录；不承诺完整微信历史，不复制 FileBrowser 文件权限。Skills Runtime、旺店通/S6、RAG、统一安装器仍为后续范围；GPT-5.6 API 是计划，第一阶段不建设独立 OCR。六阶段编号与定义见[项目总纲](../00_项目总纲.md)。
