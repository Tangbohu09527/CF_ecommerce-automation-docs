# 电商业务全自动化系统

> 当前状态日期：2026-09-04
>
> 本仓库只维护企业自动化总体架构、跨仓库状态、部署恢复边界、验证记录和技术决定，不包含业务代码、生产配置、凭证或真实业务数据。

## 项目定位

CFserver 是部署宿主；`CF_agent-gateway` 与 PostgreSQL 是消息和控制状态权威。Windows AI 主机上的 Hermes external runtime 是执行边界。员工从微信发起请求，Gateway 完成持久化、身份权限、线程、路由、响应和投递控制；正式企业文件访问未来统一经过 `CF_filebrowser-enterprise` 的 File Service、权限检查和审计。

## 当前总状态

> 企业消息与 AI 文本闭环基础已完成生产交付，项目进入文件服务、Skills 和业务系统集成阶段。

六阶段规划中的阶段 1 还包含文件基础链路，因此阶段 1 的全部退出条件尚未满足；消息 Runtime 里程碑完成不等于整个“电商业务全自动化系统”已经完成。

| 组件或能力 | 当前结论 |
| --- | --- |
| `CF_agent-gateway` | branch authority 为 `main`；2026-09-04 verified snapshot 为 `4f13039b86c60bc94340edb5468f0102d62d2dff`，PR #8/#9 docs-only closeout 已合并且 main CI 通过；production Release authority 仍为 `b488cf452584e73bc9b752564bf90ea153aa8d18` |
| `CF_agent-wechat` | branch authority 为 `main`；2026-09-04 post-promotion snapshot 为 `69f07702b6ee16d8e9700b3a53d5ebbb8ee875f8`，PR #1/#4/#5/#6 与 main CI 已完成；forced-QR 生产行为仍以 2026-09-03 验收为准 |
| PostgreSQL | Gateway 权威状态已在线，revision `20260823_04`；真实 restore 演练仍未完成 |
| Hermes | 当前文本链路真实调用成功，AI 主机重启后 reachability 曾恢复；长期 watchdog、告警和高可用未收口 |
| `CF_filebrowser-enterprise` | V1 Beta implementation and automated validation completed; CFserver deployment and production acceptance pending |
| Skills、旺店通、S6 | 尚未接入生产任务链 |

详细分层状态见[当前状态矩阵](./status/current-status.md)，本次生产事实见[2026-09-03 Enterprise Runtime Production Closeout](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。

2026-09-04 repository/documentation closeout 已完成；当前只剩本仓库 PR #7 最终人工复核。组件 `main` 的 live tip 必须动态查询，上表 SHA 只表示本次 dated snapshot，不是永久 current main。

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

私聊文本与真正结构化 `@` 机器人的群聊文本已经完成真实回复闭环；普通群消息未明确 `@` 时不会调用 AI。Gateway V2 `group_sender` 已在代码和自动化测试中按 sender identity 隔离，但“同群多个发送者互不串线”尚缺单独的生产对照验收。

## 已交付范围

- Gateway V2 四个应用进程与 PostgreSQL 的生产 Runtime、Runtime Controller 和不可变镜像发布。
- 私聊、群聊真实 `@`、未 `@` 安全结束、Response/Delivery 与 Bot self 防回环。
- Checkpoint generation、回退/rebase、历史前缀跳过、实时后缀单次处理和无重复回复。
- Admin `uncertain` Dispatch 查询与受控恢复能力的仓库实现、自动化测试和部署代码；已有一次受控生产恢复证据，但并非所有恢复动作都已生产演练。
- P1 日志降噪、Gateway `64m x 10` 日志策略、回滚 Release、离线镜像和带日期证据。
- forced fresh QR、CFserver 重启恢复、AI 主机重启后的 Hermes reachability，以及 Gateway-only 切换保持微信 Session。

## 尚未交付范围

- 完整入站文件/图片理解、Hermes 多模态、出站 Artifact 文件/图片闭环。
- 引用正文自动注入 Hermes。
- FileBrowser 的 CFserver 部署、迁移、备份恢复、WebDAV/OnlyOffice 真实联调及 Agent 主链集成。
- General AI Provider routing、Skills Runtime、企业知识库/RAG、旺店通、S6 和正式业务权限矩阵。
- PostgreSQL restore、agent-wechat automatic boot stop gate、Hermes 长期 watchdog/告警/容量/高可用。
- 独立 OCR；第一阶段按既定决定不建设该能力。

## 权威文档

| 主题 | 唯一入口 |
| --- | --- |
| 系统当前状态 | [status/current-status.md](./status/current-status.md) |
| 当前进度与下一步 | [status/current-progress.md](./status/current-progress.md) |
| 系统架构 | [architecture/system-architecture.md](./architecture/system-architecture.md) |
| 跨项目部署导航 | [deployment/deployment-guide.md](./deployment/deployment-guide.md) |
| 跨系统恢复 | [operations/recovery-runbook.md](./operations/recovery-runbook.md) |
| 生产验收清单 | [validation/production-validation-checklist.md](./validation/production-validation-checklist.md) |
| 技术决定 | [05_技术决策记录.md](./05_技术决策记录.md) |
| 2026-09-03 生产收口 | [Enterprise Runtime Production Closeout](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md) |

`docs/` 与标注日期的旧材料只作为兼容入口或历史证据；XMind 与 PNG 是早期蓝图。当前事实以代码、测试、合并提交、生产证据及上述权威文档为准。

## 相关仓库

| 仓库 | 职责 |
| --- | --- |
| `CF_ecommerce-automation-docs` | 总体架构、跨仓库状态、运维和路线图 |
| `CF_agent-gateway` | 消息、身份权限、线程、路由、Context、Dispatch、Response、Delivery 和审计控制 |
| `CF_agent-wechat` | 微信登录、消息读取与发送的外部通道 Runtime |
| `CF_filebrowser-enterprise` | 唯一正式企业 File Service 与持久审计边界 |

生产运行不得持续依赖 GitHub 在线。跨仓库事实必须按精确 SHA、PR 状态和证据日期核对；未合并组件 PR 不得写成组件 `main` 权威。
