# 当前进度与下一步

> 状态日期：2026-09-03
>
> 能力分层以[当前状态矩阵](./current-status.md)为准，生产证据以[2026-09-03 Production Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)为准。

## 当前工作流

企业消息与 AI 文本闭环基础已完成生产交付。六阶段规划仍处于阶段 1，因为原阶段定义还包含文件基础链路；当前实际工作已经从消息 Runtime 交付转向文件服务、可靠性、Skills 和业务系统集成准备。

## 已完成里程碑

- Gateway V2 production runtime 与 P1 observability。
- 私聊与真正 `@` 的群聊文本链路，以及未 `@` 安全结束。
- forced fresh QR 生产行为。
- Checkpoint regression/rebase、历史前缀与实时后缀、self skip 和无重复回复。
- CFserver 核心重启恢复与 fresh QR 重新放行。
- AI 主机重启后的 Hermes reachability 恢复。
- Gateway-only cutover、回滚 Release、离线镜像与生产证据留存。
- `uncertain` Admin recovery、Context Runtime 等 Gateway 能力的仓库实现和自动化验证。

## 正在进行

- Gateway component docs closeout：PR #8，OPEN，未合并。
- WeChat R2 component docs closeout：PR #5，OPEN，堆叠在 PR #4 之上，未合并。
- Enterprise docs closeout：本仓库 PR #7。
- WeChat PR #4 / PR #1 的 promotion 顺序、CI 失败和 main 权威收口。

未合并的组件文档 PR 只能作为 companion work，不得当作组件 `main` 的当前权威。

## 建议优先级

以下为当前建议，不是不可变承诺；技术决定变化仍以[技术决策记录](../05_技术决策记录.md)为准。

1. 完成三个仓库文档 PR 的人工复核。
2. 收口 `CF_agent-wechat` PR #4 -> PR #1 的正确提升顺序及 CI。
3. 完成 FileBrowser CFserver 部署、迁移和恢复验收。
4. 收口 Hermes watchdog、开机自启和生产监控。
5. 生产验证同群多发送者的 `group_sender` 隔离。
6. 完成引用正文上下文。
7. 完成媒体和文件双向桥。
8. 建设 Skills Runtime。
9. 接入旺店通和 S6。
10. 分批授权正式业务身份与群。

## 持续限制

- automatic boot stop gate 未完成，CFserver reboot 后 fresh QR 前必须显式检查并关闭 Gate。
- agent-wechat 自身重启、重建或 Host reboot 都需要 fresh QR；Archive 不可自动复用。
- AI 主机 reboot 与 Gateway-only deploy 通常不需要 fresh QR，但仍要完成各自健康验证。
- FileBrowser、Skills、完整媒体、RAG、旺店通和 S6 均不属于当前生产交付。
