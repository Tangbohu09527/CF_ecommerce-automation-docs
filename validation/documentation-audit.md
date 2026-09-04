# 文档审计与收口报告

> 文档编号：VAL-AUDIT-20260904
>
> 审计日期：2026-09-04
>
> 审计范围：本仓库全部 54 个 Markdown 文件

## 1. 审计边界

本次只修改 `CF_ecommerce-automation-docs` Markdown。通过用户明确授权的 GitHub API 只读查询核对组件分支、PR、提交、代码、测试、CI 和文档；未 clone、checkout、修改或访问三个组件仓库的本地工作区，也未连接 CFserver、AI 主机、Docker、数据库或生产文件。

## 2. 跨仓库事实

| 组件 | 核对结果 |
| --- | --- |
| Gateway | branch authority=`main`；2026-09-04 verified snapshot=`4f13039b86c60bc94340edb5468f0102d62d2dff`；PR #8/#9 MERGED，docs-only baselines 分别为 `c5518aed12b90235f118ed81bb3cef75d0463443` 与 `4f13039…`；main CI Run `33863057556` completed/success，3/3 Jobs 成功；production Release authority 仍为 `b488cf452584e73bc9b752564bf90ea153aa8d18` |
| WeChat | branch authority=`main`；2026-09-04 promotion baseline=`02583fe76220916019ca961bb37dfa015640384e`，post-promotion docs snapshot=`69f07702b6ee16d8e9700b3a53d5ebbb8ee875f8`；PR #1/#4/#5/#6 MERGED；main CI Run `33863104399` completed/success，5/5 Jobs 成功；observed production image provenance 仍未精确映射到 Release Commit |
| FileBrowser | `main=4750a97cfdf5bd067e04b6b36bf9616f5ada836d`；`feat/v1-integration=48380c3f31cb37b01d0c05b8db0cfa49680a17f9`；无开放 PR/Release；对应 branch CI success |

Gateway component documentation closeout、WeChat repository promotion 和 WeChat component documentation closeout 均已完成。Repository live tip 仍须动态查询；dated snapshot 和 production authority 不互相替代。

## 3. 关键事实纠正

- 当前状态入口复核至 2026-09-04；2026-09-03 Production Closeout 继续保留原 Evidence date。
- Gateway V2/P1 从“部分链路待验证”更新为生产已交付，并记录 merged main、source SHA、image digest、revision 和 Release。
- forced-QR R2 repository promotion、Fixture/CI 修复、dirty conflict 解决和 component docs closeout 已完成；真实生产行为仍以 2026-09-03 验收为准。
- CFserver reboot 更新为核心恢复、agent-wechat 保持停止、Controller `stop` 组合 Poll/Delivery Gate + fresh QR 后恢复；automatic boot stop gate 仍未验证。
- AI host reboot 更新为 Hermes reachability 恢复一次，微信 Session 保持；不外推为 watchdog/HA 完成。
- Gateway-only deployment 更新为不重建 agent-wechat、不需要 fresh QR。
- Runtime Controller v1 修正为组合 Poll/Delivery Gate：`stop/start` 始终同时作用于 `worker` 与 `delivery-worker`，不支持单 Worker 控制，也不管理 Dispatch Worker。
- Dispatch Worker 明确由 Gateway Release/Compose 生命周期独立启动和验证；组合 Gate 关闭时 Gateway/Dispatch 可以保持在线。
- `uncertain` 正式 Admin inspection/recovery API 从“缺少能力”更新为 implemented/tested/deployed，生产动作覆盖继续分层。
- FileBrowser 从笼统“开发中”更新为 V1 Beta implementation/automated validation completed，production deployment pending。
- Hermes 当前文档不再硬编码无法由本次部署证据确认的版本；历史版本仅保留在带日期记录。
- 动态 Message、Checkpoint、Queue、container、heartbeat 和 Archive 数量移出长期当前状态。

## 4. group thread 核对

GitHub 只读代码/测试核对结果：

- V1 compatibility `build_group_thread_key` 忽略 sender，按 source account + physical group conversation 形成 whole-room thread。
- V2 `ThreadResolver` 的 `group_sender` key 包含 sender identity、Profile revision 和 policy。
- Gateway 自动化测试覆盖同群不同发送者获得不同 V2 Thread。
- 生产 Runtime 使用 V2 代码线，但现有生产证据只证明群聊真实 `@` 文本闭环，没有同群两个发送者的专门对照。

最终分类为 Repository implemented / automated tests passed / deployed / same-group multi-sender production validation pending。

## 5. Context、Admin、媒体和业务能力

| 能力 | 分类 |
| --- | --- |
| Context Timeline/Snapshot/search | implemented、tested、main CI passed、deployed code；full production exercise pending |
| Admin `uncertain` recovery | implemented、tested、deployed；one controlled production recovery，all actions not fully exercised |
| media discovery | production validated to image bytes/integrity |
| full media pipeline | not production validated |
| FileBrowser | V1 Beta implementation and automated validation completed; deployment pending |
| Skills / 旺店通 / S6 | not integrated / not verified |

## 6. 文档权威层级

唯一入口已经明确：

1. `status/current-status.md`
2. `status/current-progress.md`
3. `architecture/system-architecture.md`
4. `deployment/deployment-guide.md`
5. `operations/recovery-runbook.md`
6. `validation/production-validation-checklist.md`
7. `05_技术决策记录.md`
8. `validation/records/2026-09-03-enterprise-runtime-production-closeout.md`

`docs/` 只保留兼容入口和历史快照；`design/` 顶部增加 implementation/validation/remaining scope/authority 状态头。

## 7. 历史隔离

2026-08-04、2026-08-11、2026-08-13、2026-08-14、V1、Staging 和 Bootstrap 材料保留原日期、SHA、当时限制和历史命令，并增加 Historical/Archived 或 compatibility 说明。

历史材料不能外推：

- 旧 Hermes 版本为当前版本；
- 旧 Checkpoint/消息数量为当前动态状态；
- V1 whole-room thread 为当前 V2 策略；
- 旧“未启用/未部署”覆盖当前 Gateway 生产状态；
- 旧媒体发现等于当前完整媒体能力。

## 8. 最终组件状态审计

- Gateway PR #8/#9 已合并，component documentation closeout 标记为 COMPLETED。
- WeChat PR #1/#4/#5/#6 已合并，repository promotion 与 component documentation closeout 标记为 COMPLETED。
- 两个组件 main CI Run 均 completed/success，且每个 Job 成功。
- 固定 Merge Commit 只作为 2026-09-04 dated repository snapshot 或 PR merge baseline，不标记为永久 live main。
- Gateway 生产继续由 `b488cf… / f36c798… / b9341ca…` 证据组定义，docs-only merge 不表示重新部署。
- WeChat 仓库合并未重建或部署现场镜像，observed image ID 与 Release Commit/构建输入的 exact mapping 仍未证明。
- 2026-09-03 Production Closeout 中当时组件 PR 未完成的状态属于历史证据，保持原文。
- Enterprise documentation closeout baseline completed on 2026-09-04.

## 9. 2026-09-04 定向语义审计

- 当前长期状态页不再包含组件 PR 未合并、合并冲突、失败门禁或文档收口进行中状态。
- 2026-09-03 Production Closeout 继续保留当时尚未完成提升的 PR 快照；该命中属于 historical evidence。
- `c5518aed…`、`4f13039…`、`02583fe…` 和 `69f0770…` 的全部当前命中均明确标记为 dated repository snapshot、PR merge baseline 或 promotion baseline，不标记为永久 live main。
- Gateway/WeChat branch authority 均写为 `main`，live tip 要求通过 GitHub 动态查询。
- Runtime Contract v1 没有 Poll-only、Delivery-only 或 Dispatch Worker 控制；需要单 Worker 控制时必须作为 future contract change 实现和验收。
- 最终旧状态计数：Gateway 两个文档 PR 开放状态均为 0；WeChat promotion 的开放状态为 1，且仅位于 2026-09-03 dated evidence；dirty、unstable、PR #6 开放、提升待完成和文档进行中均为 0。

## 10. 自动检查

临时 Python 检查器从标准输入运行，未写入或提交仓库。

| 检查 | 结果 |
| --- | --- |
| 严格 UTF-8 | PASS，54/54 |
| 每文件单一 H1 | PASS |
| Markdown code fence | PASS |
| Mermaid fence | PASS |
| 仓库内相对链接/目录/图片 | PASS，300 links，1 image |
| Anchor | PASS，2 anchors |
| 空链接 | PASS |
| Windows 绝对路径 | PASS |
| 内网 IP | PASS |
| 微信 ID / chatroom 模式 | PASS |
| Secret / Authorization value 模式 | PASS |
| 冲突标记 | PASS |
| 行尾空格 | PASS |

最终提交后已重新运行全量文档检查、`git diff --check`、`git log --check`、PR 状态核验和工作区洁净检查，结果均为 PASS。

## 11. 未修改范围

- 未修改 `.github/`、Workflow、代码、配置、Compose、脚本、Token、数据库或生产数据。
- 未修改 PNG、XMind 或其他二进制附件。
- 未修改、提交或推送其他仓库。
- 未连接或修改 CFserver/AI 主机。

总体规划 XMind/PNG 仍是早期蓝图；如需视觉内容与当前架构完全同步，应作为未来单独文档任务处理。
