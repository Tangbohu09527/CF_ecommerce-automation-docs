# 文档审计与收口报告

> 文档编号：VAL-AUDIT-20260903
>
> 审计日期：2026-09-03
>
> 审计范围：本仓库全部 54 个 Markdown 文件

## 1. 审计边界

本次只修改 `CF_ecommerce-automation-docs` Markdown。通过用户明确授权的 GitHub API 只读查询核对组件分支、PR、提交、代码、测试、CI 和文档；未 clone、checkout、修改或访问三个组件仓库的本地工作区，也未连接 CFserver、AI 主机、Docker、数据库或生产文件。

## 2. 跨仓库事实

| 组件 | 核对结果 |
| --- | --- |
| Gateway | `main=b488cf452584e73bc9b752564bf90ea153aa8d18`；组件 PR #7 merged；main CI success；docs PR #8 OPEN，head `75287d57c2ffa4fad7e3cd7b5ce0c175ee23cd8a` 且 checks green |
| WeChat | `main=92393bc2ae1d89dae9449fc131413979aa2fa2f2`；PR #1 和 #4 OPEN；docs PR #5 OPEN，head `ddaa7d466b6dfae6a4df8f95e11dea5a4be13b02`；PR #4/#5 当前 checks 部分失败 |
| FileBrowser | `main=4750a97cfdf5bd067e04b6b36bf9616f5ada836d`；`feat/v1-integration=48380c3f31cb37b01d0c05b8db0cfa49680a17f9`；无开放 PR/Release；对应 branch CI success |

未合并组件 PR 只作为 companion work，不作为 `main` 权威。

## 3. 关键事实纠正

- 状态日期从 2026-08-14 更新到 2026-09-03。
- Gateway V2/P1 从“部分链路待验证”更新为生产已交付，并记录 merged main、source SHA、image digest、revision 和 Release。
- forced-QR R2 从“完全新设备 QR 未验证”更新为生产行为已验证，同时保留 WeChat PR 栈尚未 main promotion 和 CI 失败。
- CFserver reboot 更新为核心恢复、agent-wechat 保持停止、显式 stop Gate + fresh QR 后恢复；automatic boot stop gate 仍未验证。
- AI host reboot 更新为 Hermes reachability 恢复一次，微信 Session 保持；不外推为 watchdog/HA 完成。
- Gateway-only deployment 更新为不重建 agent-wechat、不需要 fresh QR。
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

## 8. stale statement audit

按任务正则执行后保留 201 个命中，分布在 39 个文件。分类如下：

| 分类 | 关键保留原因 |
| --- | --- |
| Current and correct | `group_sender/private_sender`、两类日志策略、WeChat PR 状态、automatic boot stop gate 限制、FileBrowser 尚未部署 |
| Historical and intentionally retained | 2026-08 日期、Hermes historical version、V1 Worker 状态、17 Checkpoint/151 历史等 dated counts |
| Current limitation | same-group multi-sender production validation、FileBrowser deployment、PostgreSQL restore、Hermes HA、完整媒体/Skills |
| Dated evidence | Gateway/WeChat PR/SHA、Release、log capacity、Closeout 证据 |
| Stale and removed | 8 月当前状态、QR/Host/AI reboot 全未验证、Gateway 部分链路、uncertain 无 API、FileBrowser 仅普通开发中 |
| Requires component follow-up | WeChat PR #4/#1 promotion 与 PR #4/#5 CI；Gateway PR #8、WeChat PR #5 最终合并状态 |

## 9. 自动检查

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

Git `diff --check` 在每组提交前通过。最终提交后还需重新运行总 diff、commit 和 PR 检查。

## 10. 未修改范围

- 未修改 `.github/`、Workflow、代码、配置、Compose、脚本、Token、数据库或生产数据。
- 未修改 PNG、XMind 或其他二进制附件。
- 未修改、提交或推送其他仓库。
- 未连接或修改 CFserver/AI 主机。

总体规划 XMind/PNG 仍是早期蓝图；如需视觉内容与当前架构完全同步，应作为未来单独文档任务处理。
