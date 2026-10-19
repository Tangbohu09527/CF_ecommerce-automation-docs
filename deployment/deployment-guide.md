# Deployment Guide

> 中文名称：跨项目生产部署指南
>
> 文档编号：DEP-001
>
> 状态同步日期：2026-10-20。以下为通用部署边界；本次只同步文档，不执行部署或恢复。

## 1. 适用范围

本文规定企业 Runtime 的跨仓库部署顺序和门禁，不复制组件仓库的完整 Runbook。每次部署必须使用精确 commit、image digest、database revision、Compose project、回滚 Release 和脱敏验证记录。

### 当前现场证据与接续边界

PDF、PNG 及正常商品 ZIP 出站已有真实业务证据，不能再以 10 月 11 日“新计划未切换”作为当前部署断点。旧事实保留在[历史摘要](../validation/records/2026-10-11-overall-status-evidence.md)。最后现场观察为 **2026-10-19 北京时间 18:36**：收发关闭、无在途、原 Bot 登录，随后临时授权撤销、临时密码删除；本轮不访问生产，也不推断现在运行状态。

当前正式机器仍使用项目插件和受限客户端；Gateway、FileBrowser 服务端、Windows 客户端、Hermes、WeChat 镜像及报告提交是不同身份，完整值见[本轮固定证据](../validation/records/2026-10-20-overall-status-evidence.md)。以后的部署先核对当时状态和正式发布输入，不重放旧 plan 或重复安装相同工具补证。

### 最终部署标准与候选边界

- 服务端一条正式、可复用的入口配合必要参数完成部署，并保持各组件资源归属、固定源码/制品、权限与恢复边界。
- 电脑只安装官方 Hermes，再配置少量参数即可运行；无需额外项目 EXE、手工复制插件、修改 Hermes 源码或项目专用常驻进程。
- 不依赖 Codex 工作树、`.task-artifacts` 或旧现场回执。
- 临时运维权限撤销后，正式业务凭业务凭据继续运行；管理权限撤销本身不证明业务持续运行已经验收。

新机简部署尚未满足最终标准。另一个 Gateway 任务负责统一服务端入口和官方 Hermes 纯配置接入，**开发验证中**；本轮不接管其本地文件、分支或 PR，不等它完成才交付文档。FileBrowser [PR7](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/7) 已推送固定组件候选，验证首次准备中断后的恢复/拒绝边界；不是完整 Debian 联调或生产部署证明。

MCP 是验证路线，不能写成已迁移或生产已采用。[Gateway PR23 固定报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/1f3a1c2978bcc0b4803e6246e309fc77a7865c51/docs/validation/2026-10-20-native-http-mcp.md) 已证明不改官方源码、无项目插件/EXE/PYTHONPATH 的纯配置发现、正式图文工具消息及真实 TLS FileBrowser fixture 的合成大 ZIP 查询/导出。两个业务会话复用一个 MCP transport session、每调用元数据为空，缺可信 Gateway task/session/employee 绑定，实验哨兵返回 `trusted_task_context_missing`。尚未进入 Artifact/Response/微信协议替身交付及身份错配/过期/跨会话/重放矩阵。专用 Ubuntu CI 成功不等于全新 Debian 部署/重启；其他 CI 截止仍运行中。完整授权/撤销、任务绑定和业务链须继续验证，再单独安排获准现场验收。官方 Hermes 原版接入、无项目专用本机进程、独立新机可重建及管理权限撤销后的业务连续性须分别验收。组件既有 clean-device/compatibility CI 不自动证明这些新标准已经全部满足。

时区默认值候选与运行配置分别核对：Gateway PR22、WeChat PR10、FileBrowser PR6/7 未合并；报告中的实际时区调整不等于候选镜像已运行。按批准策略分别记录宿主、应用日志、数据库存储及连接时区，不沿用旧“全部容器 UTC”的一刀切结论。

## 2. 发布输入

| 输入 | 要求 |
| --- | --- |
| 文档 | 本仓库精确 commit |
| Gateway | 当前 main、获准候选/source SHA、合并状态、实际 Image ID、可独立核实的 registry digest 与目标 schema 分列；当前产品 PR 栈尚未合并，不能写成 main |
| WeChat | 实际批准分支/SHA、immutable image；未完成 main promotion 时明确写 PR 栈 |
| PostgreSQL | image、revision、备份与 restore 计划 |
| Hermes | external runtime 的批准控制和健康检查；不从历史文档猜版本 |
| FileBrowser | 已有同机部署和受限文件操作证据；继续核对实际 image、候选合并状态、权限与恢复范围，不能由旧待部署条目否定已有现场进展 |
| Secret | 只写受控引用，不记录值、hash/prefix 或完整环境文件 |

未解析占位符、漂移 `main`、`latest` 镜像或缺失回滚目标都会阻断部署。

## 3. 基础环境

- CFserver 宿主、应用和数据库时区按本次批准的策略分别记录；数据库存储与日志显示口径不能混同，固定报告与候选见本轮摘要。
- Docker、存储挂载、日志空间、`cf-internal` 和离线镜像已准备。
- PostgreSQL、Gateway、agent-wechat 各自使用明确 ownership。
- Windows AI 主机具备受控 Hermes 启停与健康检查。
- 不记录内网 IP、主机账号、Secret 或 Windows 本地绝对路径。

## 4. 标准部署顺序

### 4.1 Host、Docker、storage、network

只读核对时间、Docker、挂载、容量和 `cf-internal`。网络已存在但定义不匹配时停止，不通过删除并猜测重建修复。

### 4.2 PostgreSQL

恢复数据库 readiness，核对备份与目标 revision。只有 reviewed migration 可以改变 schema；普通 restart 不运行 migration。

### 4.3 Gateway core 与 Dispatch Worker

先核对目标 schema 与当前 Release 的兼容性，仅在获准的迁移窗口执行需要的 migration；普通重启不默认运行 upgrade。Gateway API 与 Dispatch Worker 分别验证 readiness/heartbeat。Dispatch Worker 由 Gateway Release/Compose 生命周期负责，不属于 Runtime Controller 的 controlled services；组合 Poll/Delivery Gate 关闭时，Gateway API 与 Dispatch Worker 可以保持在线。生产长期应用进程以 `10001:10001` 运行，使用 immutable image 和 Gateway 专属 `64m x 10` 日志策略。

### 4.4 Runtime Controller

核对 `contract`、`status`、ready 和 Token contract。Runtime Contract v1 的 controlled services 只有 Poll Worker（`worker`）和 Delivery Worker（`delivery-worker`）：Controller `stop` 同时停止二者，`start` 同时准备、重建并启动二者。当前没有 Poll-only、Delivery-only 或 Dispatch Worker 控制。

### 4.5 agent-wechat Bootstrap

Bootstrap 只检查 Docker/Compose/目录/权限/Token mount/网络，不登录、不恢复 Session、不放流。生产契约必须保持：

- `restart: "no"`；
- loopback 6174；
- external `cf-internal`；
- `ENABLE_VNC=0`；
- `20m x 3`；
- Token read-only；
- Archive 不自动复用。

### 4.6 fresh QR

1. 检查 Controller status。
2. 通过 Controller `stop` 显式关闭组合 Poll/Delivery Gate，不假设 Host boot 后已 stopped。
3. 使用组件仓库批准的唯一 forced-QR 入口。
4. 手机扫码。
5. 验证 WeChat 进程、auth、chats、messages 和容器/API health。
6. 失败时保持 Gate 关闭并保留证据。

QR、Cookie、Session、Token、账号和 Chat ID 不进入仓库或普通日志。

### 4.7 Hermes

按批准方式启动或核对 Hermes external runtime，从 CFserver 验证 reachability。不要把 Worker liveness 当成 Hermes connectivity。

### 4.8 打开组合 Poll/Delivery Gate

Dispatch Worker 已在 Gateway Release/Compose 生命周期中独立启动并验证。fresh QR 与 Hermes 门禁全部通过后，通过 Controller `start` 同时准备、重建并启动 Poll Worker 与 Delivery Worker。Poll 只入队 durable Dispatch，Dispatch Worker 独立调用 Hermes，Delivery 只处理持久化 Response/Outbox。

当前如需单独启动或停止 Poll/Delivery 中任一服务，必须作为 Runtime Contract future change 实现和验收，不能从本指南推断已经支持。

### 4.9 status

必须同时核对：

- database/revision；
- Gateway readiness；
- Dispatch Worker heartbeat，以及组合 Gate 内 Poll/Delivery 两个 Worker 的 heartbeat；
- WeChat auth/API；
- Hermes connectivity；
- Controller ready 与 Token contract；
- Dispatch/Delivery backlog 和 `uncertain`；
- Checkpoint continuity。

### 4.10 production validation

执行[生产验证清单](../validation/production-validation-checklist.md)，失败或未运行项不得被健康检查替代。

## 5. 四类变更

| 场景 | fresh QR | 核心动作 |
| --- | --- | --- |
| CFserver reboot | 必须 | agent-wechat 保持停止；Controller stop 组合 Gate，核对独立 Dispatch，再 fresh QR 和 Controller start |
| Gateway-only deploy | 不需要 | 不重建 agent-wechat，保持 Session |
| agent-wechat restart/recreate | 必须 | Controller stop 组合 Gate、Archive、fresh Runtime、QR、API 验证、Controller start |
| AI host restart | 通常不需要 | Session 保持；验证 Hermes reachability 与 Queue |

## 6. 升级与回滚

- 先保存当前 SHA、digest、revision、Controller status、日志和 backlog。
- Gateway 回滚到 previous immutable Release；不覆盖 active Release。
- 应用回滚和数据库回滚分开，旧应用必须与当前 schema 明确兼容。
- agent-wechat 代码/镜像变化仍需 stop -> Bootstrap -> fresh QR。
- 不执行 `docker compose down -v`、跨项目 `down`、`--remove-orphans` 或 volume 删除。
- 不删除数据库行、Checkpoint、Queue、Response、Outbox、Archive 或恢复审计。

## 7. FileBrowser 已有部署与后续门禁

FileBrowser 已有同机部署、受限 API 与客户端操作、一个商品 4 主图/8 详情归档并保留原包的实际证据；正式 Hermes 商品工具加载和正常微信 ZIP 出站实收由后续 Gateway 报告补齐。源码、服务端运行镜像、插件/客户端与报告提交分别见[本轮摘要](../validation/records/2026-10-20-overall-status-evidence.md)，不能用早期“未测微信”的报告段落覆盖后续完成结论。

以下按各自证据继续核验，不能视为全部未开始或全部已完成：

1. Candidate/immutable image。
2. shared-host ownership 与网络。
3. migration。
4. database/config/business file backup。
5. isolated restore。
6. rollback。
7. real WebDAV and OnlyOffice。
8. File Service permission/audit。
9. Gateway/Hermes integration。

FileBrowser 保持唯一正式文件系统，Gateway/Hermes 通过 API、权限与审计接入；正式原件与工作副本分开。PDF/PNG 和正常 ZIP 出站的限定验收保留；ZIP 入站、更多资料整理、共享群收件和通用加工回存继续分项推进。小批量资料整理与文件类 Skill 可并行安排，路径调整明确暂缓；最终简部署由独立候选按本页标准验证。

## 8. 禁止事项

- 把一个项目的 Compose 当成另一个项目的所有者。
- 打印 `.env`、Token、Authorization 或 database URL。
- 把未合并候选或组件文档 PR 写成 main/已部署，或把旧故障手册起点当作最新现场断点。
- 用旧 Session 或 Archive 跳过 fresh QR。
- 用手工数据库写入恢复 `uncertain`。
- 把 FileBrowser 限定部署扩大为完整文件业务验收，或把企业 Skills Runtime、独立 OCR、旺店通/S6 的规划写成已经上线。

具体故障流程见[Recovery Runbook](../operations/recovery-runbook.md)。
