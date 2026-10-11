# 电商业务全自动化系统

> 同步日期：2026-10-11。仓库与现场证据分别记录；本次只更新 Markdown，不部署、不恢复服务、不执行业务验收。

## 整体目标

员工微信 → 微信入口 → Gateway 权限/上下文/任务 → 官方 Hermes 与工具 → 企业文件服务/后续业务系统 → 结果返回微信。CFserver 是 Debian 部署宿主，Gateway/PostgreSQL 保存消息与控制权威状态，Windows AI 主机承担 Hermes 和执行工具。

AI 与员工共用 `CF_filebrowser-enterprise` 这一唯一正式文件系统。主线继续沿用 [9 月 27 日文件业务需求](./requirements/file-workflows.md)：内容分类、指定目录归档、查找并回传微信、下载工作副本加工后通过 API 回存；正式原件默认保留。旺店通、S6 等仍属后续集成。

## 当前能做什么，卡在哪里

| 问题 | 截至 2026-10-11 的结论 |
| --- | --- |
| 已有能力证据 | 私聊/真正 @ 群聊文本有历史闭环；**2026-09-17** 分钟级任务持久化与微信回传通过。**09-25 至 09-27** 已有 FileBrowser 同机部署、受限接入、真实文件读取和受控文本新建的限定现场证据 |
| 当前接续点 | 用户回传：旧 f9 两端回退、d9 `complete-rollback` 和旧核心恢复完成；随后 d9 新候选镜像、两端新 plan 通过，**尚未切换**。新 deployment 尚未 `apply/start/verify`，原聊天附件实收未验收 |
| 最新未知 | 用户随后已执行 CFserver 重启；**重启后状态未核验**，Windows 是否关机未确认。重启前 Hermes/Gateway API/Dispatch 运行、Poll/Delivery 停止、Gate 关闭和数据库正常，不证明重启后已恢复 |
| 下一步 | 先核验重启后现场与现存新计划，符合条件再接续切换；不从头回退、传包或构建。当前先交付微信真实文件/图片收发，不并行扩安装器或大重构 |

“已有能力证据”不表示此刻在线可用。六阶段规划保持不变：阶段 1 的消息 Runtime 有历史交付，文件基础链路尚未完成，不给虚构完成百分比。

完整 SHA、PR、CI、现场候选和截止点统一见 [2026-10-11 脱敏证据摘要](./validation/records/2026-10-11-overall-status-evidence.md)，分层结论见[四项目状态矩阵](./status/current-status.md)。三个组件的固定候选与各自 main 分开核对，未合并分支不写成 main；组件其他文档任务不作为本次已完成成果。

## 文件业务验收边界

以下六项分别留证：微信入站取得字节、内容理解、指定目录/分类归档、文件中心查询回传、原聊天实际附件实收、加工结果回存。文本新建、读取成功、HTTP 200、CI 绿不能相互替代，缩略图/元数据也不证明取得原件。细则见[验收清单](./validation/production-validation-checklist.md#b11-媒体文件和业务扩展)。

正式文件只能通过 FileBrowser API、权限与审计访问；入站媒体仍遵守 Gateway 持久化，出站仍要求 READY Artifact。预授权资料群常规新增自动归档是目标，群/身份/目录规则未确定前不全局放行，未知分类进入待确认。AI 主机离线时 CFserver 先收原件、随后分类仍待实现/验收。

## 后续规划

- 轻量 Gateway Web：考虑局域网 IP 访问、Gateway 权限管理、查询 Gateway **已经接收的部分**聊天/任务记录；尚待规划，不承诺微信完整历史，不复制 FileBrowser 文件权限体系。
- 共享群分类归档与工作副本加工沿用既有需求，接续真实文件/图片收发推进。
- Skills Runtime、旺店通/S6、RAG、统一安装器和可靠性专项按证据及原六阶段后续安排；GPT-5.6 API 仍是计划，第一阶段不建设独立 OCR。

## 权威入口

| 主题 | 入口 |
| --- | --- |
| 文件业务需求 | [requirements/file-workflows.md](./requirements/file-workflows.md) |
| 当前能力与证据分层 | [status/current-status.md](./status/current-status.md) |
| 当前断点与下一步 | [status/current-progress.md](./status/current-progress.md) |
| 最新脱敏证据 | [2026-10-11 总体状态摘要](./validation/records/2026-10-11-overall-status-evidence.md) |
| 系统架构 | [architecture/system-architecture.md](./architecture/system-architecture.md) |
| 部署与恢复 | [部署导航](./deployment/deployment-guide.md)、[恢复手册](./operations/recovery-runbook.md) |
| 分项验收 | [validation/production-validation-checklist.md](./validation/production-validation-checklist.md) |
| 六阶段与固定决定 | [项目总纲](./00_项目总纲.md)、[技术决策记录](./05_技术决策记录.md) |
| 历史限定成功 | [2026-09-17 长任务回传](./validation/records/2026-09-17-gateway-long-task-acceptance.md) |
| 历史总体基线 | [2026-09-03 Production Closeout](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md) |

## 仓库边界

| 仓库 | 职责 |
| --- | --- |
| `CF_ecommerce-automation-docs` | 总体架构、跨仓库状态、阶段、部署恢复边界及带日期证据 |
| `CF_agent-gateway` | 身份权限、上下文、任务、文件引用、消息和投递控制及审计关联 |
| `CF_agent-wechat` | external 微信通道 Runtime 与媒体适配 |
| `CF_filebrowser-enterprise` | AI 和员工共用的唯一正式文件系统、API、权限与持久审计 |

本次基于 `8179533ffade1027d01727cbf5a89a26b491c400`，文档分支为 `codex/docs-overall-status-20261011`，Draft PR base 为 `docs/file-business-workflows-20260927`，保留[原 Draft PR #9](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/9)。本次不合并、不打 Tag。

生产运行不得持续依赖 GitHub 在线。源码、CI、Image ID、registry digest、完整构建来源和真实业务验收不能互相推导。`docs/`、旧状态、XMind 和 PNG 按日期作为历史材料。
