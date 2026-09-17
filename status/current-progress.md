# 当前进度与下一步

> Gateway 工作面更新：2026-09-17。其他组件沿用各自带日期基线，本轮没有执行全项目状态复核。
>
> 能力分层以[当前状态矩阵](./current-status.md)为准；最新 Gateway 证据见[2026-09-17 长任务回传](../validation/records/2026-09-17-gateway-long-task-acceptance.md)，此前总体交付见[2026-09-03 Production Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。

## 当前工作流

阶段 1 的消息 Runtime 已有生产交付，文件基础链路仍未完成。此次先恢复微信生产入口和长任务回传，不重写架构；本轮限定场景已通过，转入文档收口与残余事项跟踪，不重复部署或重发已通过的测试。

## 本轮完成

- Gateway PR #11 修复代码已合并，固定合并基线已通过 GitHub 核验。
- 现场新 Image ID、Dispatch 启动 600 秒等待配置及停止宽限期已核对。
- 既有 uncertain 任务通过独立带审计的 mark_dead 终止，保留原错误与审计；既有排队只读任务完成回传。
- Dispatch 与 Controller 管理的 Poll/Delivery 经受控步骤恢复。
- 新的分钟级任务：服务端派发 113.799 秒，响应落库、一次投递及回执与微信实收匹配。
- 原始工具日志、历史告警、制品来源及未验收专项的边界明确保留。

本次 Gateway 文档 PR #12 在记录时 OPEN；本仓库同步固定提交摘要。文档提交/PR 创建不表示已合并 main，也不表示再次部署。

## 历史已完成里程碑

以下保持原证据日期，不作为本轮重新执行的项目：Gateway V2/P1、9 月 4 日 Gateway/WeChat component documentation closeout、forced-QR R2 repository promotion、私聊与真正 @ 的群聊、未 @ 安全结束、Checkpoint regression/rebase/self skip、CFserver 核心重启及 fresh QR、AI host reachability、Gateway-only Session 保持，以及 Context/Admin recovery 等的仓库实现和自动化验证。

## 当前工作面与建议顺序

1. 审查并合并此次两个独立文档 PR，保留 CI、commit 和来源日期；合并需另行确认。
2. 单独核对 Hermes 原始工具日志/安装构建与现场镜像来源，保全新版离线证据和回滚材料；不得为补证重跑带副作用任务。
3. 对历史会话连续性告警分别取证和处理，不通过删队列/改 Checkpoint 消除告警；按授权安排断线、并发、续租及重启专项。
4. 继续原定 FileBrowser CFserver 部署/迁移/恢复验收，再推进 Hermes watchdog/监控、群聊双发送者隔离、引用正文、媒体与文件双向桥。
5. 文件与权限边界稳定后建设企业 Skills Runtime，再接入旺店通/S6。

以上为建议次序，不是已安排的自动任务或已获批准的生产变更；技术决定仍以[技术决策记录](../05_技术决策记录.md)为准。

## 持续限制

- automatic boot stop gate 未完成，CFserver reboot 后 fresh QR 前须显式关闭组合 Gate。
- agent-wechat 自身重启/重建与 Host reboot 的 fresh QR 规则不变，Archive 不自动复用。
- AI host-only reboot 与未触碰微信入口的 Gateway-only 切换不因此强制 fresh QR，但仍须验证健康；本次不新增无人值守恢复承诺。
- FileBrowser、企业 Skills、完整媒体、RAG、旺店通、S6 不因一次本地文件工具测试而成为已交付。
- 新 Image ID 不等于已核实 registry digest 或完整 source-to-image provenance。
- 一次 113.799 秒成功不等于 600 秒上限、执行中断线/重启、全部会话、watchdog 或高可用验收。
