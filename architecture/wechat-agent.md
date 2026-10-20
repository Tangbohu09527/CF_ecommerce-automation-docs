# agent-wechat 定位与职责

> 需求同步日期：2026-10-20。最后现场观察为 2026-10-19 北京时间 18:36，见[固定证据摘要](../validation/records/2026-10-20-overall-status-evidence.md)；本轮未访问生产，历史验收保留原日期。

## 当前定位

`CF_agent-wechat` 是外部微信通道 Runtime，不是 Gateway 内部模块。它负责微信进程、fresh QR、auth/chats/messages、消息读取和发送；Gateway/PostgreSQL 负责身份权限、线程、Dispatch、Response 和 Delivery 权威状态。

## 已有运行契约

最新后置[恢复目标](../requirements/confirmed-platform-scope.md)要求统一入口检查/恢复缺失服务，微信在线保留登录、离线再提示扫码。下表的 forced-QR 重启契约是现有实现，与新目标冲突处尚未调整；本轮不改脚本、不做恢复或新机试验。

| 项目 | 当前值 |
| --- | --- |
| Compose project / container | `cf-agent-wechat` |
| restart | `"no"` |
| API | 6174，loopback only |
| network | external `cf-internal`，alias `cf-agent-wechat` |
| VNC | `ENABLE_VNC=0`，无 VNC/noVNC/x11vnc/websockify |
| logs | `json-file 20m x 3` |
| Token | 独立只读 mount |
| Session | Host/container/Runtime restart 后 fresh QR；Archive 不复用 |

## 2026-09-03 历史已验证行为

- forced fresh QR、手机扫码与 auth/chats/messages 放行。
- 私聊和群聊文本读取/发送。
- 图片来源发现与 JPEG 字节读取；没有证明该 JPEG 是源原图。
- CFserver reboot 后保持停止，fresh QR 后重新上线。
- Gateway-only deployment 不重建容器并保持 Session。

## 历史 Git 与镜像边界

- 2026-10-11 GitHub 只读核对：`main=81d21f451adc61df71077ca3035adeddeca4621f`，由 [PR #7](https://github.com/Tangbohu09527/CF_agent-wechat/pull/7) 于 09-06 合并；对应两项 Actions 成功。
- 媒体候选 `codex/fix-wechat-original-media=9b997ac92d653380a01ac93e0ac65e4a0e3a4c0a` 比 main 多 2 个提交，未合并、未发现对应 PR；三个候选 Actions 成功。完整 run、SHA 和截止点见[10 月 11 日摘要](../validation/records/2026-10-11-overall-status-evidence.md)。
- 候选仅加入固定上游来源锁、隔离验证、CI 与文档，未修改生产 Compose/安装脚本；来源核验不能证明已切换运行镜像。
- 09-04 promotion/documentation 与当时 observed Image ID 属于历史；见[09-03 总体 Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)，不把旧镜像称为本次新候选。

以上固定提交保留其历史日期，不作为当前远端 tip 或现场镜像。缓存契约隔离测试不替代真实微信验收；后续 PDF/PNG 样本与正常 ZIP 出站实收已有独立证据，不能继续写成全部未验收。PNG 图片业务已被接受，不再追查原字节差异。合法原文件名仍属必须解决的通道问题，已有候选/隔离验证与实际协议部署及收件验收分开，不能降为提示词优化。详细历史组件边界见[固定媒体核验文档](https://github.com/Tangbohu09527/CF_agent-wechat/blob/9b997ac92d653380a01ac93e0ac65e4a0e3a4c0a/docs/validation/2026-09-28-media-retrieval-candidate.md)，最新汇总见[状态矩阵](../status/current-status.md)。

## 2026-10-11 历史现场截止点

用户回传旧回退及旧核心恢复已完成，新候选与两端新 plan 已通过，尚未切换；新 deployment 未 `apply/start/verify`，附件实收未验收。随后用户已执行 CFserver 重启，重启后微信、Gate 与其他服务状态均未重新核验，Windows 是否关机未确认。重启前正常与历史 reboot 验收不能证明此次已恢复，不能沿用历史 PID。

这段是当日断点；10 月 19 日现场观察已有后续记录，不再把此处当作本轮下一步。本轮仅文档同步，不执行切换、回退、登录或 Gate 操作。

## 不负责

- Enterprise Identity、Access Policy、Admission 和 Thread Policy。
- Hermes 调用、Context、Skills 和业务系统。
- Message/Checkpoint/Dispatch/Response/Delivery 权威状态。
- 正式企业文件权限和 Persistent Audit。

## 当前需求与限制

- automatic boot stop gate 不由本组件保证，且尚未验证。
- agent-wechat 自身 restart 后不能复用旧 Session。
- 已有 PDF/PNG 与正常 ZIP 出站限定验收，不等于通用 ZIP 入站和所有文件场景完成。
- Archive 和备份长期保留由外部运维策略负责。
- 平台需持续采集聊天档案；当前 Gateway 轮询 self 跳过、窗口/范围/停止及连续性缺口保留。未 `@` 可留档而不执行，普通附件不处理不影响消息/元数据留档。
- 按任务取得必要原件，明确保存才归档，纯文档群明确配置可自动收件；不把所有群附件下载永久保存，不继续安排离线自动补分类专项。

底层实现以组件仓库和 PR 栈为准；系统级操作见[Recovery Runbook](../operations/recovery-runbook.md)。
