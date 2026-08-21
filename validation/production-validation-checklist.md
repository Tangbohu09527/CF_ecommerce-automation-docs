# Production Validation Checklist

> 中文名称：生产验证清单
>
> 文档编号：VAL-001
>
> 清单版本：1.0
>
> 文档状态：执行模板
>
> 最近复核：2026-08-21

## 1. 使用规则

本清单用于新部署、升级、回滚和恢复演练。每次执行复制为一份新的 `validation/records/<UTC_DATE>-<release_id>-production-validation.md` 记录；不得在本模板上直接填生产结果。

- 每项初始为 `NOT_RUN`。
- `PASS` 必须包含预期、实际、脱敏证据引用和复核人。
- `FAIL` 阻断发布。
- `BLOCKED` 必须说明缺少的能力，并从获准生产范围中排除。
- 历史 PASS 不自动继承到新 commit、镜像、配置、schema、Profile 或主机。
- 不使用真实员工消息、业务文件或生产正文作为测试夹具。

## 2. 执行记录头

| 字段 | 执行时填写 |
| --- | --- |
| `record_id` | `<UTC_DATE>-<release_id>-production-validation` |
| `checklist_version` | `1.0` |
| 环境 / `release_id` / change ticket |  |
| UTC 开始/结束时间 |  |
| `Asia/Shanghai` 开始/结束时间 |  |
| 操作者 / 复核人 / 批准人 |  |
| 文档仓库 commit |  |
| Gateway commit/tag / image digest |  |
| WeChat commit/tag / image digest |  |
| Gateway/WeChat Compose path、project、config hash |  |
| PostgreSQL image digest / schema / migration |  |
| Hermes version / package hash / `external_profile_ref` |  |
| 备份 ID / 恢复命令版本 |  |
| 批准 time source / `max_clock_skew` / 测量方法 |  |
| 测试数据说明 | 仅脱敏测试账号、群和内容 |

任一版本字段缺失时，不开始会改变生产状态的测试。

## 3. 发布前检查

| ID | 检查项 | 预期 | 结果 | 证据 |
| --- | --- | --- | --- | --- |
| PRE-01 | 发布清单完整且无占位符 | commit、digest、Compose、schema、Hermes、回滚均已解析 |  |  |
| PRE-02 | Secret 管理 | 仓库、普通 YAML、命令输出和截图中无 Secret |  |  |
| PRE-03 | 备份 | 数据库和必要状态已备份，恢复目标与命令明确 |  |  |
| PRE-04 | `cf-internal` | 仅获准组件连接，Token 鉴权仍启用 |  |  |
| PRE-05 | Host 时区 | Debian 为 `Asia/Shanghai`；Windows 为 `China Standard Time` |  |  |
| PRE-06 | Container 时区 | 每个生产容器实测为 UTC |  |  |
| PRE-07 | Database 时区 | PostgreSQL `SHOW timezone` 为 UTC |  |  |
| PRE-08 | Application 转换 | 固定 UTC 样例正确显示为 `Asia/Shanghai` 并可往返 |  |  |
| PRE-09 | 时钟同步 | CFserver、AI 主机、数据库漂移低于发布阈值 |  |  |
| PRE-10 | 基线健康 | 数据库、Gateway、三个 Worker、微信和 Hermes 达到发布定义 |  |  |

## 4. Restart 测试

### 4.1 故障注入安全门禁

Restart、daemon、数据库和宿主重启都是故障注入，不得把本节项目按编号在生产机械顺序执行。优先在隔离 Staging、恢复演练或等价隔离环境完成；确需在生产执行时，每个场景必须独立满足并留证：

- 逐项变更审批、明确影响范围和维护窗口。
- 备份已经证明可在隔离环境恢复，而不只是“备份命令成功”。
- 回滚负责人、执行负责人、停止条件和升级联系人明确。
- 入口流量冻结或严格限定为批准的脱敏测试流量，并定义积压处理方式。
- fault injection 命令、目标服务/主机和预期故障均已批准；禁止临场猜测命令。
- 监控、证据采集和用户影响通知已经就绪。

任一门禁不满足时，该场景记录 `BLOCKED` 或 `NOT_RUN`，不得为了填完清单继续执行。

### 4.2 分层测试

每个层级单独执行并留证；低层级 PASS 不替代高层级。

| ID | 场景 | 核心预期 | 结果 | 证据 |
| --- | --- | --- | --- | --- |
| RST-01 | Gateway 应用服务 restart | 从原 Checkpoint/线程恢复，无丢失、重复或手改 DB |  |  |
| RST-02 | Gateway 容器 recreate | volume、网络、schema、Checkpoint 和绑定恢复 |  |  |
| RST-03 | PostgreSQL restart | readiness 恢复，Worker 重连，权威状态完整 |  |  |
| RST-04 | Docker daemon restart | 两套 Compose、网络、volume 和启动顺序恢复 |  |  |
| RST-05 | CFserver reboot | Docker、数据库、微信 session、Workers 和积压恢复 |  |  |
| RST-06 | Windows AI 主机 reboot | Hermes 自动启动、健康、Profile 和 CFserver 连通恢复 |  |  |
| RST-07 | `agent-wechat` 容器 restart | 现有 session 恢复，不重新 bootstrap，不丢消息 |  |  |
| RST-08 | 重启窗口内积压 | 每个进入执行链的获准来源事实只形成一个逻辑 Dispatch；允许留下受控 attempt，但不得产生重复可见回复或业务副作用 |  |  |

当前证据只覆盖 RST-01 的有限场景。其余项目在目标能力未完成时记录 `BLOCKED`，不能预填 PASS。

## 5. 微信登录与 QR 测试

| ID | 场景 | 核心预期 | 结果 | 证据 |
| --- | --- | --- | --- | --- |
| WLG-01 | 现有有效 session | 批准企业 Bot、Source Account Mapping、session 路径、Checkpoint namespace 和目标会话与发布清单一致；无需 QR 即可验证登录/API |  |  |
| WLG-02 | 现有 session 失效后 QR 恢复 | 仅用目标版本批准脚本登录同一企业 Bot；映射和 namespace 不变，不重新 bootstrap，积压按保留窗口恢复 |  |  |
| WLG-03 | 完全新设备 QR | 覆盖脚本、手机确认、session 持久化、restart 和首次采集；当前无实机证据，必须记录 `BLOCKED` 或 `NOT_RUN`，不得预填 PASS |  |  |
| WLG-04 | 错误账号或映射不一致 | 登录后发现 Bot、Source Account Mapping、Checkpoint namespace 或目标会话不一致时阻断采集与 Dispatch，不修改映射迁就错误账号 |  |  |
| WLG-05 | session restart 持久化 | `agent-wechat` restart 后 session 按批准位置恢复，仍绑定同一 Bot/Source Account；Checkpoint 连续且不重新 QR/bootstrap |  |  |

二维码、Cookie、session 和真实账号不得进入记录、日志、截图或附件。完全新设备 QR 通过新的实机记录前，保持当前未验证边界。

## 6. 消息测试

| ID | 场景 | 核心预期 | 结果 | 证据 |
| --- | --- | --- | --- | --- |
| MSG-01 | 获准私聊文本 | Persist-first、Admission Allowed、原私聊唯一回复 |  |  |
| MSG-02 | 未获准私聊 | 消息持久化、拒绝、不调用 Hermes、不回复 |  |  |
| MSG-03 | 获准群聊真实 `@` | `group_sender`、独立线程、原群唯一回复 |  |  |
| MSG-04 | 群聊未 `@` | `bot_not_mentioned`、不调用 Hermes、不回复 |  |  |
| MSG-05 | 纯文本伪 `@` / 引用但未 `@` | 不从正文或引用推断 mention |  |  |
| MSG-06 | Bot `is_self=true` | sink 前过滤、推进 Checkpoint、不回环 |  |  |
| MSG-07 | 引用类型消息 | `reply_context` 持久化；仅按当前能力验证文本回复 |  |  |
| MSG-08 | 图片来源 | 类型、Raw Payload、字节、签名/大小/hash；未接入节点为 BLOCKED |  |  |
| MSG-09 | 文件与多附件 | 中文名、类型、大小、重复和顺序；能力未完成时 BLOCKED |  |  |
| MSG-10 | 重复 `localId` | 同一 `sourceAccount + chatId + localId` 只形成一个来源事实 |  |  |
| MSG-11 | 持久化前中断 | Checkpoint 不越过，恢复后重新读取 |  |  |
| MSG-12 | 持久化后/Checkpoint 前中断 | 允许重读，唯一键吸收，不产生第二个 Dispatch |  |  |
| MSG-13 | API 分页/短时断线 | 不越过未知消息，恢复后连续读取 |  |  |
| MSG-14 | 上游保留窗口越过 | 明确记录同步缺口，不宣称零丢失 |  |  |
| MSG-15 | 已有环境恢复 | 不执行 `bootstrap_mode=latest`，沿用原 Checkpoint |  |  |

## 7. Hermes 测试

| ID | 场景 | 核心预期 | 结果 | 证据 |
| --- | --- | --- | --- | --- |
| HER-01 | 版本/健康/Profile | 版本和安装包匹配；Profile 引用可解析 |  |  |
| HER-02 | 获准文本 Dispatch | 只调用一次，绑定预期 Thread/Profile，形成明确 Response |  |  |
| HER-03 | 未获准消息 | Hermes 侧没有对应调用证据 |  |  |
| HER-04 | Hermes 不可达 | Dispatch 进入明确失败或 `uncertain`，不伪造回复 |  |  |
| HER-05 | `uncertain` | 禁止自动盲重试，正式命令/API、Guard 和审计通过 |  |  |
| HER-06 | 守护与告警 | 进程退出被发现，告警到达，按批准方式恢复 |  |  |
| HER-07 | 入站媒体 | Attachment 经受控协议进入 Hermes；未实现时 BLOCKED |  |  |
| HER-08 | Skills | 权限、确认、幂等和审计通过；未实现时 BLOCKED |  |  |

## 8. 回复测试

| ID | 场景 | 核心预期 | 结果 | 证据 |
| --- | --- | --- | --- | --- |
| REP-01 | Response Persistence | Hermes 结果先持久化，再创建 Outbox |  |  |
| REP-02 | 私聊回复 | Outbox/Attempt 与原私聊关联，实际只收到一次 |  |  |
| REP-03 | 群聊回复 | Outbox/Attempt 与原群关联，不发到其他会话 |  |  |
| REP-04 | Delivery 临时失败 | 保留 Outbox/Attempt，受控重试后不重复送达 |  |  |
| REP-05 | Response 已存在但未送达 | 只恢复 Delivery，不创建第二个 Hermes Dispatch |  |  |
| REP-06 | Bot 防回环 | 回复被 Polling 识别为 self，不产生新 AI 工作 |  |  |
| REP-07 | 媒体回复 | Artifact 原子物化、完整性和 `READY` 后才能投递；未实现时 BLOCKED |  |  |

## 9. 数据验证

| ID | 检查项 | 核心预期 | 结果 | 证据 |
| --- | --- | --- | --- | --- |
| DAT-01 | 端到端关联 | 同一脱敏 ID 串联 Message、Admission、Thread、Routing、Dispatch、Response、Outbox、Attempt |  |  |
| DAT-02 | 来源幂等 | `message.id`、来源 ID、`event_id` 分离，重复来源不重复入库 |  |  |
| DAT-03 | Checkpoint | 每来源账号+会话隔离；非 self 持久化后推进 |  |  |
| DAT-04 | 线程隔离 | 私聊/群聊隔离，同群不同发送者不串线；`group_shared` 未批准时不可用 |  |  |
| DAT-05 | Profile 所有权 | Gateway 只保存 `external_profile_ref`，不创建 Hermes 档案 |  |  |
| DAT-06 | 时间 | 持久时间 UTC；展示转换一次且正确；日志可跨主机关联 |  |  |
| DAT-07 | 审计与日志 | 有状态、耗时、错误分类和脱敏关联 ID；无正文/Secret/真实账号 |  |  |
| DAT-08 | 备份恢复 | 目标备份可恢复到隔离环境，schema/计数/关键关联一致 |  |  |
| DAT-09 | 媒体元数据 | 数据库只存元数据，不长期存大文件 Base64；未实现时 BLOCKED |  |  |
| DAT-10 | 正式文件 | 只经 File Service、最小权限和审计；未接入时 BLOCKED |  |  |

## 10. 结果汇总

| 项目 | 执行时填写 |
| --- | --- |
| PASS 数量 |  |
| FAIL 数量 |  |
| BLOCKED 数量及排除范围 |  |
| NOT_RUN 数量及理由 |  |
| 已确认消息丢失/重复/错投 |  |
| 残余风险与 owner |  |
| 回滚是否执行 |  |
| 最终决定 | `GO` / `NO-GO` / `LIMITED-GO` |

`LIMITED-GO` 必须逐项列出不可用能力、访问范围和到期复核时间。任一安全不变量、数据一致性、消息错投、重复业务副作用或 Secret 泄漏失败只能是 `NO-GO`。

## 11. 签核

| 角色 | 姓名/标识 | 决定 | 时间 | 备注 |
| --- | --- | --- | --- | --- |
| 执行人 |  |  |  |  |
| 技术复核 |  |  |  |  |
| 运维批准 |  |  |  |  |
| 业务/安全批准（按需） |  |  |  |  |
