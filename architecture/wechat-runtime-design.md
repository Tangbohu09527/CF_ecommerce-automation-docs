# WeChat Runtime Design

> 中文名称：微信运行时设计
>
> 文档编号：ARC-002
>
> 文档状态：当前基线
>
> 需求同步日期：2026-10-20；已有运行契约与最新恢复目标并列，实机证据保留原日期。

最新固定状态见[2026-10-20 总体证据](../validation/records/2026-10-20-overall-status-evidence.md)，最后现场观察为 10 月 19 日北京时间 18:36，本轮未访问生产。旧候选与重启断点保留在[10 月 11 日历史记录](../validation/records/2026-10-11-overall-status-evidence.md)。代码、隔离测试、部署与业务验收分别判断。

## 1. 目标

微信 Runtime 负责发现来源消息、保留来源事实、推进 Checkpoint，并在 forced-QR、进程中断和来源 Local ID 回退后避免历史重放与重复回复。它不负责身份授权、Hermes 执行或业务系统操作。

## 2. 当前组件

```mermaid
flowchart LR
    WX["微信"] <--> AW["external agent-wechat"]
    AW --> PW["Gateway Poll Worker"]
    PW --> MS["Message Store"]
    PW --> CP[("Checkpoint generation")]
    MS --> AD["Admission / V2 Dispatch"]
    CTL["Runtime Controller"] --> PW
    CTL --> DLW["Gateway Delivery Worker"]
```

agent-wechat 与 Gateway 属于两个独立项目和 Compose ownership。两者通过 `cf-internal` 与 Token contract 通信。

## 3. 已有 forced fresh QR 契约与新目标

最新[用户确认](../requirements/confirmed-platform-scope.md)要求后置的统一恢复入口检查并恢复缺失服务，微信在线保留登录、离线再提示扫码，分别显示采集/AI 执行/投递状态；不默认清 runtime、重装、升级、迁移或每次强制 fresh QR。下面是现有契约，与新目标冲突部分尚未调整实现，本轮不修改运行脚本，不执行恢复。

- agent-wechat 使用 `restart: "no"`。
- Host、container 或 Runtime restart 后必须 fresh QR。
- 旧 Runtime 在新启动前归档；Archive 不能挂回 active Session。
- 生产不使用 VNC/noVNC/x11vnc/websockify，`ENABLE_VNC=0`。
- API 6174 只绑定 loopback，并通过 `cf-internal` alias 提供给 Gateway。
- fresh QR 后必须验证进程、auth、chats 和 messages，随后才能恢复 Gate。

forced-QR R2 repository promotion 与 component documentation closeout 已于 2026-09-04 完成，branch authority 为 `main`，main CI 全部成功。仓库合并没有重新构建或部署生产镜像；forced-QR 真实生产行为仍以 2026-09-03 验收为准。

## 4. Polling 与 Persist-first

- 每个来源账号与物理会话独立维护 Checkpoint。
- self reply 在 sink 前跳过，但仍推进 Checkpoint。
- 非 self 消息先持久化，再执行 Admission 和 V2 Routing。
- Poll Worker 只入队 durable Dispatch，不直接调用 Hermes。
- 首次 `LATEST` 建基线；已有环境恢复禁止重新 bootstrap。

用户要求 Bot 在线期间持续采集私聊/群聊，人工 Web 与机器人复用同一 Gateway 档案。上面的 self 跳过是现存缺口：自身出站另有 Response/Delivery 不代表 Bot 其他自发消息均归档，防回环应阻止 AI 触发而非永远阻止留档。Poll 停止、范围限制、上游窗口和连续性也会影响完整性；未 `@` 可以归档但不执行，普通附件不处理不等于不存消息元数据。Poll 是采集，历史日期查询读取已归档记录，不重新轮询。

## 5. Local ID 与 generation

Local ID 是来源事实，不假设跨 Session 单调。forced QR 后发生回退时，Runtime 使用：

- generation；
- server-ID-first 或 content-free continuity anchor；
- checkpoint fingerprint；
- compare-and-swap rebase；
- generation-scoped fallback identity。

安全 rebase 将当前可见历史前缀作为新基线，不调用 Message Sink；随后实时后缀只处理一次。连续性无法证明时 fail closed，不手工降低 Checkpoint。

## 6. 2026-09-03 已完成生产验收

- forced-QR 后 Local ID 回退。
- Checkpoint generation。
- 历史前缀跳过。
- 实时后缀只处理一次。
- 私聊普通前进。
- 空窗口实时后缀。
- self reply skip。
- 无重复回复。
- Queue 与业务链一致。

动态 Chat、Checkpoint 和历史消息数量仅属于带日期证据，不是长期配置。

## 7. Host reboot

2026-09-03 真实 CFserver reboot 中：

- Docker 与 Gateway 核心恢复；
- agent-wechat 保持停止；
- 旧 Session 未恢复；
- Poll/Delivery 曾被观察为 running/healthy。

因此 automatic boot stop gate 未验证。Operator 必须先检查状态并通过 Runtime Controller `stop` 同时停止 Poll Worker 与 Delivery Worker，再运行 forced-QR 入口；验证完成后只能用 `start` 同时恢复二者。Dispatch Worker 由 Gateway Release/Compose 生命周期独立管理。

10 月 11 日重启未知状态和此前完成的回退/恢复保留原记录，不再当作当前操作断点。10 月 19 日实际观察及后续候选见[10 月 20 日固定证据](../validation/records/2026-10-20-overall-status-evidence.md)。本轮不核验在线、不执行旧计划；恢复仍不得盲重做结果不明的历史任务。

## 8. AI host 与 Gateway-only 变更

- AI host reboot 不重启 agent-wechat 时 Session 保持，不需要 fresh QR；恢复后核对 Hermes reachability。
- Gateway-only deployment 不重建 agent-wechat，Session 保持，不需要 fresh QR。
- 两者都不能外推为 agent-wechat restart 可以复用 Session。

## 9. 日志与证据

- agent-wechat：`json-file 20m x 3`。
- Gateway：`json-file 64m x 10`。
- 不记录 QR、Token、账号、会话 ID、消息正文或 Archive payload。
- 一次性 Archive/Release 路径只进入带日期 Closeout。

## 10. 未完成边界

- automatic boot stop gate。
- 上游长期消息保留窗口验证。
- 选定 Release Commit、构建输入与现场 Image ID 的 provenance。
- 保留 PDF/PNG 样本和正常 ZIP 出站限定验收；ZIP 入站、合法原文件名协议部署/收件和一般保存后检索仍分项有缺口。
- Archive 长期保留和容量策略。
- 上游 API/schema 升级复验。

生产证据见[2026-09-03 Production Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。
