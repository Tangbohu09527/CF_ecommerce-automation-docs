# 电商业务全自动化系统

> 需求同步：2026-10-20。本轮仅文档、必要只读代码核对与文档检查。最后生产观察仍为 **2026-10-19 北京时间 18:36**，本轮未访问生产、未重新验收。

## 当前目标

员工微信 → 微信入口 → Gateway 权限、分组、会话与 Profile 路由 → 官方 Hermes 和获准工具 → 唯一正式 File Service → 结果返回微信。四仓先完成平台基础，业务理解、商品别名/分类/手卡/模板及电商流程交后续 Hermes/Skills。保持原六阶段编号。

本轮[最新确认正文与需求差异表](./requirements/confirmed-platform-scope.md)是当前需求入口；[D044—D053](./05_技术决策记录.md)说明与 D026、D040—D043 等旧决定的关系。

- **B 方案已确认：** 分组是模板，每员工、每微信群各自拥有独立真实 Hermes Profile；同组不共享具体 Profile 状态。员工授权与分组分离，允许已授权未分组；群发起员工另须群使用权限。
- **电脑按使用规模分配：** 在 Web 将分组明确分配给目标电脑，独立 Profile 在对应电脑运行；不定机器数，不强制 UI 另机，不默认自动负载均衡或运行中迁移。
- **Profile 管理目标已确认，可行性待验证：** Gateway 计划受控创建及同步模板派生 Profile。保存引用/版本、真实路由、创建、同步、旧会话生效五层分开；本轮不做实验。
- **当前基础还包括：** 同私聊/群持续会话、执行中补充、当前群必要上文/引用/日期查询、聊天采集与授权查询、合法原文件名。现有 FIFO 或 Profile 字段不代表这些都完成。
- **文件默认不主动处理：** 元数据留档、任务临时取得原件、正式归档分开；按任务取得、明确要求才保存，指定纯文档群可配置自动保存例外。
- **Skills 后置：** 四仓基础完成后用户找 Hermes 制作并另建仓库；旧并行试点不再是当前待办。简部署/恢复保留最终要求并后移；AI 离线自动补分类专项撤出，已有持久化和防重复保留。

## 已有成果与实际缺口

| 已有证据 | 保留的限制 |
| --- | --- |
| PDF 正文读取/原件回传、PNG 看图及图片实收、同会话文字接续 | 原样回传不等于按要求保存再独立检索；PNG 原字节差异按 D041 不再追查 |
| 单商品 4 主图/8 详情归档并保留原包；受限查询、复制、移动、改名、替换、可恢复删除/还原 | 非全库整理，API/客户端成功不等于每项真实模型选用已验收 |
| 正常微信 ZIP 出站，整包及 12 成员与收件匹配 | ZIP 入站历史超时仍失败，完整入站未验收；合法原名候选尚未协议部署/收件验收 |
| Gateway Message、时间字段、档案读取/管理 API；线程和任务状态基础 | self 在 Poll 跳过、出站分存、采集窗口/连续性有缺口；现 Context 工具只覆盖成功 AI 轮次，非全部当前群上文；Web/管理写接口/资源采集仍缺 |
| PR12 后续统一入口与官方 Hermes 完整依赖实验 | 纯配置可信每任务授权返回链、真实新机/统一三组件 booted host/整机重启仍缺；不得沿用旧 PR23 部分结论覆盖最终证据 |

现状分层见[当前状态](./status/current-status.md)，当前安排见[当前进度](./status/current-progress.md)。历史样本、失败、固定 SHA 和 CI 原始结果完整保留于 [PR11 总体证据](./validation/records/2026-10-20-overall-status-evidence.md)与 [PR12 最终候选证据](./validation/records/2026-10-20-configuration-only-candidate.md)。本轮没有功能完成或生产在线的新声明。

## 权威入口

| 主题 | 入口 |
| --- | --- |
| 最新确认及差异表 | [平台基础、B方案与业务边界](./requirements/confirmed-platform-scope.md) |
| 原六阶段 / 功能 / 决定 | [总纲](./00_项目总纲.md)、[功能需求](./01_功能需求.md)、[技术决策记录](./05_技术决策记录.md) |
| 当前状态与后续细化 | [状态矩阵](./status/current-status.md)、[进度](./status/current-progress.md) |
| 文件场景 | [文件工作流](./requirements/file-workflows.md) |
| 架构 / 部署 / 恢复 / 验收 | [系统架构](./architecture/system-architecture.md)、[部署指南](./deployment/deployment-guide.md)、[恢复手册](./operations/recovery-runbook.md)、[验收清单](./validation/production-validation-checklist.md) |
| 本轮基线与代码核对 | [需求同步记录](./validation/records/2026-10-20-requirements-sync.md) |

## 仓库与合并边界

| 仓库 | 职责 |
| --- | --- |
| `CF_ecommerce-automation-docs` | 总需求、阶段、决定、跨仓状态与验收导航；本轮唯一修改/提交仓库 |
| `CF_agent-gateway` | 消息、权限、会话/Profile 路由、档案、任务、投递与审计关联 |
| `CF_agent-wechat` | 微信登录、消息收发与附件协议适配 |
| `CF_filebrowser-enterprise` | 唯一正式文件系统、服务端权限与持久审计 |

本轮分支 `codex/docs-confirm-profile-b-20261020` 从 PR11 准确 HEAD `462ecc3a16f92e843b8c2276c828fa7b3d159599` 建立，Draft PR base 为 `codex/docs-overall-status-20261020`。按语义吸收 PR12 `3aa7bbbbe3ec3a1d756b06bdc5ed2d0265f6a041` 的最终候选证据与状态；PR11 的总体需求、D040—D043、验收和恢复导航保留后由新增决定明确调整。

远端核对时 main 为 `dd81075d7516d74a9e307ebdc03ce8ec07e12c49`，PR9—PR12 均 OPEN/Draft。后续合并依赖为 PR9 → PR10 → PR11 → 本轮；PR12 与 PR11 同基于 PR10，旧 PR 保留且其新增证据已吸收，后续如单独合并需审阅重叠。本轮不关闭、合并或改旧 PR base；**Draft 更新不等于 main 同步。**

无文档 workflow。本轮检查结果见[同步记录](./validation/records/2026-10-20-requirements-sync.md)；不运行产品测试、模型、安装或现场验收。生产运行不持续依赖 GitHub 在线。
