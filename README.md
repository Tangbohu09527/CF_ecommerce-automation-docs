# 电商业务全自动化系统

> 同步日期：2026-10-20。[本轮跨仓候选与脱敏证据](./validation/records/2026-10-20-configuration-only-candidate.md)是当前导航。历史验收按原观察时间保留；本轮只做候选开发、隔离验证和文档同步，不访问生产。

## 整体目标

员工微信 → 微信入口 → Gateway 权限/上下文/任务 → 官方 Hermes 与工具 → 企业文件服务/后续业务系统 → 结果返回微信。CFserver 是 Debian 部署宿主，Gateway/PostgreSQL 保存消息与控制权威状态，Windows AI 主机承担 Hermes 和执行工具。

AI 与员工共用 `CF_filebrowser-enterprise` 这一唯一正式文件系统。主线继续沿用 [9 月 27 日文件业务需求](./requirements/file-workflows.md)：内容分类、指定目录归档、查找并回传微信、下载工作副本加工后通过 API 回存；正式原件默认保留。旺店通、S6 等仍属后续集成。

## 当前能做什么，卡在哪里

| 问题 | 截至 2026-10-20 的结论 |
| --- | --- |
| 已有能力证据 | PDF、同会话文字接续、PNG看图与图片实收通过；单商品实际分类归档与限定文件整理已有记录；10月19日正常微信商品ZIP出站整包及12成员与收件一致 |
| 当前接续点 | 独立候选实现Gateway统一服务器入口、FileBrowser/WeChat最小部署适配，验证固定官方Hermes纯配置接入；实际结果、准确SHA和CI见本轮记录，不等于生产已切换 |
| 最新未知 | 生产最后观察为10月19日；本轮没有访问当前服务、登录、队列或上游消息。微信ZIP正常入站完整验收仍缺；文件名候选未生产验收，正式归档路径调整暂缓 |
| 下一步 | 补齐统一入口新机证据及官方Hermes可信每任务授权返回链；资料库小批整理和首批文件Skill准备可并行。不重新执行已通过的PDF/PNG或重派历史任务 |

“已有能力证据”不表示此刻在线可用。六阶段规划保持不变：阶段1的消息Runtime有历史交付，完整文件业务与最终简部署标准仍有缺口，不给虚构完成百分比。

完整 SHA、PR、CI、候选和截止点统一见 [2026-10-20 脱敏证据摘要](./validation/records/2026-10-20-configuration-only-candidate.md)，分层结论见[四项目状态矩阵](./status/current-status.md)。[10月11日记录](./validation/records/2026-10-11-overall-status-evidence.md)作为历史保留，不覆盖后续业务成功。固定候选与各自 main 分开核对，未合并分支不写成 main。

## 文件业务验收边界

以下六项分别留证：微信入站取得字节、内容理解、指定目录/分类归档、文件中心查询回传、原聊天实际附件实收、加工结果回存。文本新建、读取成功、HTTP 200、CI 绿不能相互替代，缩略图/元数据也不证明取得原件。细则见[验收清单](./validation/production-validation-checklist.md#b11-媒体文件和业务扩展)。

正式文件只能通过 FileBrowser API、权限与审计访问；入站媒体仍遵守 Gateway 持久化，出站仍要求 READY Artifact。预授权资料群常规新增自动归档是目标，群/身份/目录规则未确定前不全局放行，未知分类进入待确认。AI 主机离线时 CFserver 先收原件、随后分类仍待实现/验收。

## 后续规划

- 资料库小批整理和首批文件Skill准备可与部署收敛并行；已有工具、业务规则及缺验收项见当前进度。
- 统一服务器入口与官方Hermes纯配置接入已有独立候选，仍需分层补齐授权返回和新机验收，不把当前机器修通当最终交付。
- 群聊/多人交互、完整聊天档案查询和群发继续放最后；Gateway Web本轮不开发，不复制FileBrowser权限体系。
- 旺店通/S6、RAG和可靠性专项按原六阶段后续安排，第一阶段不建设独立OCR。

## 权威入口

| 主题 | 入口 |
| --- | --- |
| 文件业务需求 | [requirements/file-workflows.md](./requirements/file-workflows.md) |
| 当前能力与证据分层 | [status/current-status.md](./status/current-status.md) |
| 当前断点与下一步 | [status/current-progress.md](./status/current-progress.md) |
| 最新脱敏证据 | [2026-10-20 统一部署与纯配置候选](./validation/records/2026-10-20-configuration-only-candidate.md) |
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

本轮文档候选为[Draft PR12](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/12)，基于PR10固定头 `0fe71a67bdc8331df9830ad69247674169b03a9d`。保留原PR9/PR10；不合并、不打Tag。旧分支、候选和正式版本分别见带日期记录。

生产运行不得持续依赖 GitHub 在线。源码、CI、Image ID、registry digest、完整构建来源和真实业务验收不能互相推导。`docs/`、旧状态、XMind 和 PNG 按日期作为历史材料。
