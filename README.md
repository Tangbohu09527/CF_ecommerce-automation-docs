# 电商业务全自动化系统

> Gateway 现场证据更新：2026-09-17。其他组件仍按各自注明的历史日期记录，本次未重新审查所有组件。
>
> 本仓库只维护企业自动化总体架构、跨仓库状态、部署恢复边界、验证记录和技术决定，不包含业务代码、生产配置、凭证或真实业务数据。

## 项目定位

CFserver 是部署宿主；`CF_agent-gateway` 与 PostgreSQL 是消息和控制状态权威。Windows AI 主机上的 Hermes external runtime 是执行边界。员工从微信发起请求，Gateway 完成持久化、身份权限、线程、路由、响应和投递控制；正式企业文件访问未来统一经过 `CF_filebrowser-enterprise` 的 File Service、权限检查和审计。

## 当前总状态

企业消息与 AI 文本闭环基础已有生产交付；2026-09-17 又完成一次分钟级任务的 Gateway 持久化及微信回传验收。历史会话连续性问题与未验收的可靠性专项仍保留。

六阶段规划中的阶段 1 还包含文件基础链路，因此阶段 1 的全部退出条件尚未满足。局部文件读取测试不等于企业 File Service、Skills、媒体或 ERP 已接入，也不等于所有会话和重启/高可用已验收。

| 组件或能力 | 带日期的结论 |
| --- | --- |
| `CF_agent-gateway` | 2026-09-17 核对 main 的修复合并基线 `9a1caa237a9053678c80f68fdb15d351d5bfecf8`（PR #11）；现场四个应用使用新 Image ID，Dispatch 启动 read/execution 为 600/600 秒；本次派发服务端耗时 113.799 秒，响应及微信实收匹配 |
| `CF_agent-wechat` | 本轮 Gate 恢复检查通过、微信登录与投递有现场证据；仓库 snapshot 仍沿用 2026-09-04 的 `69f07702b6ee16d8e9700b3a53d5ebbb8ee875f8`，本次未重新核验其 live main 或镜像来源映射 |
| PostgreSQL | 本轮 database/migration runtime 检查为 ok；Gateway 仓库 head 为 `20260823_04`；真实 restore 演练仍未完成 |
| Hermes | 本次长任务结果经 Gateway 回到微信；原始工具日志、Desktop 构建版本、watchdog/执行中断线或重启等仍未独立验收 |
| `CF_filebrowser-enterprise` | 沿用 2026-09-04 基线：V1 Beta 实现与自动化验证完成，CFserver 部署/生产验收待办；本次未核对其最新进度 |
| Skills、旺店通、S6 | 沿用已记录边界：未接入正式生产任务链；本次不改变该结论 |

详细分层见[当前状态矩阵](./status/current-status.md)。最新 Gateway 证据见
[2026-09-17 长任务回传记录](./validation/records/2026-09-17-gateway-long-task-acceptance.md)；
[2026-09-03 Production Closeout](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md)
保留为历史基线，不再作为新 Gateway 镜像的当前发布记录。

仓库 branch authority 仍为 `main`，live tip 必须动态查询。PR #11 代码已合并，Gateway 本次文档同步 PR #12 在记录时仍为 OPEN；本仓库通过固定文档提交引用证据，不将未合并的文档分支写成组件 main。此次文档同步本身不部署、不发布 Tag、不改生产状态。

## 当前生产文本链路

```text
员工微信
  -> CF_agent-wechat
  -> Gateway Poll Worker
  -> PostgreSQL Message / Admission / Dispatch
  -> Dispatch Worker
  -> Hermes external runtime
  -> Gateway Response / Delivery Outbox
  -> Delivery Worker
  -> CF_agent-wechat
  -> 微信回复
```

私聊文本与真正结构化 `@` 机器人的群聊文本已有历史真实回复闭环；普通群消息未明确 `@` 时不会调用 AI。V2 `group_sender` 的多发送者隔离有实现/自动化证据，但同群多发送者的独立生产对照验收仍待完成。

## 已记录的交付范围

- Gateway V2 四应用进程、PostgreSQL 权威状态、Controller 及历史 P1 生产交付。
- 私聊、群聊真实 `@`、未 `@` 安全结束、Response/Delivery 与 Bot self 防回环的历史验收。
- Checkpoint generation/rebase、历史前缀跳过、实时后缀单次处理和 P1 日志降噪的历史验收；不等于所有连续性异常已修复。
- Admin recovery 的仓库实现/测试；本次又补充一次带审计的 `mark_dead` 现场恢复。
- 本次分钟级任务的有限等待、响应持久化、单次投递及微信实收证据。
- forced QR、CFserver 核心重启恢复、AI host reachability、Gateway-only Session 保持的带日期历史记录。

## 尚未交付或未独立验收

- 独立历史会话连续性问题的关闭；原始工具日志与 Desktop 构建版本复核。
- 本次新版接近 600 秒、执行中断线/重启、并发/FIFO/续租和整机自动恢复专项。
- 新镜像完整来源证明、registry manifest digest、新离线归档及备份恢复/回滚验收。
- 入站文件/图片理解、Hermes 多模态、出站 Artifact 闭环及引用正文自动注入。
- FileBrowser 的部署、迁移、备份恢复、WebDAV/OnlyOffice 实际联调与 Agent 主链集成。
- Provider routing、Skills Runtime、知识库/RAG、旺店通、S6 和正式业务权限矩阵。
- PostgreSQL restore、automatic boot stop gate、Hermes 长期 watchdog/告警/容量/高可用。
- 独立 OCR：第一阶段按既定决定不建设。

## 权威文档

| 主题 | 入口 |
| --- | --- |
| 当前状态 | [status/current-status.md](./status/current-status.md) |
| 当前进度 | [status/current-progress.md](./status/current-progress.md) |
| 系统架构 | [architecture/system-architecture.md](./architecture/system-architecture.md) |
| 部署导航 | [deployment/deployment-guide.md](./deployment/deployment-guide.md) |
| 跨系统恢复 | [operations/recovery-runbook.md](./operations/recovery-runbook.md) |
| 验收清单 | [validation/production-validation-checklist.md](./validation/production-validation-checklist.md) |
| 技术决定 | [05_技术决策记录.md](./05_技术决策记录.md) |
| 最新 Gateway 限定验收 | [2026-09-17 长任务回传](./validation/records/2026-09-17-gateway-long-task-acceptance.md) |
| 历史总体生产基线 | [2026-09-03 Production Closeout](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md) |

`docs/`、旧状态记录、XMind 和 PNG 按各自日期作为兼容或历史材料，不作为实时状态。

## 相关仓库

| 仓库 | 职责 |
| --- | --- |
| `CF_ecommerce-automation-docs` | 架构、跨仓库状态、运维和路线图 |
| `CF_agent-gateway` | 消息、身份权限、线程、路由、Context、Dispatch、Response、Delivery 和审计 |
| `CF_agent-wechat` | 外部微信通道 Runtime |
| `CF_filebrowser-enterprise` | 唯一正式企业 File Service 与持久审计边界 |

生产运行不得持续依赖 GitHub 在线。代码基线、观察到的 Image ID、完整构建来源和实机验收是不同证据层；不得互相替代。
