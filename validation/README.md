# Validation 文档索引

> 文档状态：当前验证入口
> 最近复核：2026-08-21
> 最新运行证据截止：2026-08-14

本目录把“验收标准”“执行记录”和“历史证据”分开。设计存在、服务 healthy 或旧版本 PASS 都不能替代目标发布的执行证据。

## 当前入口

| 文档 | 用途 |
| --- | --- |
| [Production Validation Checklist](./production-validation-checklist.md) | 每次部署、升级和恢复演练使用的生产验收清单 |
| [文档审计与收口报告](./documentation-audit.md) | 2026-08-21 全仓文档缺口、过期项与收口结果 |
| [执行记录规范](./records/README.md) | 新验证记录的命名、证据字段和不可变规则 |
| [历史证据索引](./history/README.md) | 2026-08-14 及更早验证材料的适用范围 |
| [当前状态矩阵](../status/current-status.md) | 当前能力状态、未验证边界和下一动作 |

## 当前生产证据基线

截至 2026-08-14，已有证据支持：

- 获准私聊文本完整闭环。
- 群聊真实 `@` 的 `group_sender` 文本闭环，以及未 `@` 的 `bot_not_mentioned`。
- 未授权拒绝、Bot self 防回环、Workspace/线程隔离。
- 引用识别、`reply_context` 持久化和引用类型文本回复；引用正文尚未注入 Hermes。
- 图片发现、JPEG 字节提取和完整性校验；媒体 AI 与回传链路未完成。
- CFserver Gateway 应用服务 restart 后的有限恢复；更高恢复层级未验证。
- Hermes 不可达现象和一次带 Guard 的受控人工恢复；正式通用恢复工具未完成。

权威证据是[2026-08-14 私聊、群聊及媒体验证记录](../status/2026-08-14-private-group-media-validation.md)。该记录没有完整的 commit、镜像 digest、Compose hash、schema 和逐测试证据字段，因此只能按其日期和叙述范围使用，不能作为未来发布的可复现证明。

## 状态口径

| 状态 | 含义 |
| --- | --- |
| `PASS` | 目标发布在目标环境执行通过，具有可定位的脱敏证据与复核 |
| `FAIL` | 已执行但实际结果不满足预期；发布被阻断 |
| `BLOCKED` | 前置能力、工具或环境不存在，无法执行；发布范围必须排除该能力 |
| `NOT_RUN` | 尚未执行；不能推断结果 |
| `NOT_APPLICABLE` | 经批准确认不适用于本次范围，并记录理由 |

只有 `PASS` 可以支撑“已验证”。`BLOCKED`、`NOT_RUN`、设计文档、代码存在或旧环境结果都不能转写为成功。

## 未关闭生产门禁

- Hermes 可靠自启、守护、告警和 AI 主机重启恢复。
- 正式 `uncertain` Dispatch 管理命令/API。
- 引用正文注入 Hermes。
- Attachment、私有媒体、Hermes 多模态、Artifact `READY` 和媒体投递。
- 完全新设备 QR 登录。
- Gateway 容器 recreate、PostgreSQL、Docker daemon、CFserver 宿主重启。
- `group_shared`、File Service 主链、Skills、旺店通和 S6。

这些项目通过新记录验收前，必须保持 `BLOCKED` 或 `NOT_RUN`，不得从目标架构图推断完成。
