# 电商业务全自动化系统

> 文档同步：2026-10-20。组件证据通过 GitHub 固定提交只读核对；最后现场观察为 **2026-10-19 北京时间 18:36**，本轮未访问生产、未重新验收。

## 整体目标

员工微信 → 微信入口 → Gateway 权限/上下文/任务 → 官方 Hermes 与工具 → 企业文件服务/后续业务系统 → 结果返回微信。CFserver 是 Debian 部署宿主，Gateway/PostgreSQL 保存消息与控制权威状态，Windows AI 主机承担 Hermes 和执行工具。AI 与员工共用 `CF_filebrowser-enterprise` 唯一正式文件系统。

主线保留 [9 月 27 日文件业务需求](./requirements/file-workflows.md)：资料查询和微信交付、内容分类与指定目录归档、工作副本加工后通过 API 回存，默认保留正式原件。六阶段定义不变，不编造总体完成百分比。

## 当前能做什么，卡在哪里

| 问题 | 截至本轮可核对证据的结论 |
| --- | --- |
| PDF 与连续会话 | PDF 正文实际读取、附件返回与样本收件摘要一致；后续文字在同一会话接续 |
| PNG | **PNG图片识别及微信图片收发已验收，满足当前业务要求。** 原始字节差异等历史事实保留，不再作为开发阻塞 |
| 商品资料 | 正式商品图/ZIP 工具已安装并由正式 Hermes 接口确认加载；一个商品分类归档成功，4 张主图、8 张详情，保留原包 |
| 文件操作 | 查询、复制、移动、改名、替换、可恢复删除及还原有实际受限客户端/API 证据；不等于每项均由真实模型选择并实测 |
| 正常微信 ZIP 出站 | 从企业文件系统重新导出后完成微信实收，18,167,063 字节、12 成员，整包和成员摘要匹配；不是转发刚上传的原包 |
| 仍有限制 | ZIP 入站未完整验收；此前超时失败保留；文件名仍有前缀和中文替换，候选未生产验收；一个商品不代表全库已整理 |
| 最后现场观察 | 10 月 19 日北京时间 18:36 收发关闭、无在途、原 Bot 登录；随后临时授权撤销、临时密码删除。这是报告时间点，不能推断现在在线 |
| 当前下一步 | 简部署与官方 Hermes 纯配置接入由另一个 Gateway 任务开发验证中；可安排小批量资料整理与首批文件类 Skill 试点；现有归档路径调整由用户明确暂缓 |

10 月 11 日的回退、新计划未切换与重启后未知保留在[历史记录](./validation/records/2026-10-11-overall-status-evidence.md)，已不再是当前待执行步骤。固定版本、PR/CI、源码与实际镜像边界见[本轮跨仓证据摘要](./validation/records/2026-10-20-overall-status-evidence.md)及[当前状态矩阵](./status/current-status.md)。

## 当前部署与最终标准

当前正式机器仍使用项目插件和受限客户端。最终目标是：服务端一条正式、可复用入口配合必要参数；电脑只安装官方 Hermes，再配置少量参数即可运行，无额外项目 EXE、手工复制插件、Hermes 源码修改或项目专用常驻进程；不依赖 Codex 工作树、`.task-artifacts` 或旧现场回执；撤销临时运维权限后，业务依靠正式业务凭据继续运行。

新机简部署尚未达到最终标准。MCP 是验证路线，隔离工具发现成功不能代替整条文件业务验证，也不表示生产已经迁移。新推送的 [Gateway PR23 固定报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/1f3a1c2978bcc0b4803e6246e309fc77a7865c51/docs/validation/2026-10-20-native-http-mcp.md) 已证明官方原版 Hermes 纯配置的远程发现、图文工具消息和隔离 FileBrowser 合成 ZIP 查询/导出；仍缺可信的每任务授权绑定，未进入 Artifact/Response/微信交付。具体门槛见[部署指南](./deployment/deployment-guide.md)。Codex 开发模型改为 6.1 Sol，不代表生产 Hermes 模型或版本改变。

## 后续安排与边界

首批文件类 Skill 可从商品资料查询/图包交付、素材归档、资料完整性检查开始；已有工具、业务规则缺口和现场验收分别记录，不必等待 ERP、全部部署、新 RAG 或自研 Skills Runtime。详见[当前进度](./status/current-progress.md)。可靠性、ERP、平台集成、工作副本加工回存及全企业资料库治理仍按原证据边界推进。

Gateway Web 仍是后续规划。以下三项排在项目最后，属于需求，不是已交付事实：

- 群聊会话及多人/连续消息交互。
- AI 查询获准的完整聊天档案，按当前场景、问题和任务主动检索相关上文，不仅限于 Hermes 自身历史或成功任务；授权档案完整不等于手机微信所有历史均已采集。
- Gateway 向获准人员/群聊群发消息及附件。

## 权威入口

| 主题 | 入口 |
| --- | --- |
| 当前能力与证据分层 | [当前状态矩阵](./status/current-status.md) |
| 下一步与试点 | [当前进度](./status/current-progress.md) |
| 固定跨仓证据 | [2026-10-20 摘要](./validation/records/2026-10-20-overall-status-evidence.md) |
| 文件需求 | [文件业务场景](./requirements/file-workflows.md) |
| 部署与恢复 | [部署指南](./deployment/deployment-guide.md)、[恢复手册](./operations/recovery-runbook.md) |
| 分项验收 | [生产验证清单](./validation/production-validation-checklist.md) |
| 原六阶段与决定 | [项目总纲](./00_项目总纲.md)、[技术决策记录](./05_技术决策记录.md) |
| 架构 | [系统架构](./architecture/system-architecture.md) |
| 历史成果 | [9 月 17 日长任务](./validation/records/2026-09-17-gateway-long-task-acceptance.md)、[9 月 3 日基线](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md) |

## 仓库与合并边界

| 仓库 | 职责 |
| --- | --- |
| `CF_ecommerce-automation-docs` | 总体需求、阶段、跨仓状态、部署恢复边界及带日期证据 |
| `CF_agent-gateway` | 身份权限、上下文、任务、文件引用、消息投递和审计关联 |
| `CF_agent-wechat` | 微信通道 Runtime 与媒体适配 |
| `CF_filebrowser-enterprise` | 唯一正式文件系统、API、权限与持久审计 |

本轮从 PR10 准确 HEAD `0fe71a67bdc8331df9830ad69247674169b03a9d` 建立 `codex/docs-overall-status-20261020`，Draft PR base 为 `codex/docs-overall-status-20261011`，依赖[PR9](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/9)和[PR10](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/10)。核对时 main 仍为 `dd81075d7516d74a9e307ebdc03ce8ec07e12c49`，尚未包含这三轮文档更新。**Draft 分支更新不等于 main 已同步；本轮不合并。**

无文档 CI；本轮只做文档检查，不运行产品测试。生产运行不持续依赖 GitHub 在线。源码、CI、现场 Image ID、registry digest、文档提交与真实业务验收分别记录。
