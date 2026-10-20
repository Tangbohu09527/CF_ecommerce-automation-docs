# Deployment Guide

> 中文名称：跨项目生产部署指南
>
> 文档编号：DEP-001
>
> 状态同步日期：2026-10-20。以下为通用部署边界；本次只同步文档，不执行部署或恢复。

## 1. 适用范围

本文规定企业 Runtime 的跨仓库部署顺序和门禁，不复制组件仓库的完整 Runbook。每次部署必须使用精确 commit、image digest、database revision、Compose project、回滚 Release 和脱敏验证记录。

本轮用户已将简部署和统一恢复入口后置：当前先完成四仓平台基础，纯配置 MCP 或新机安装不再作为唯一主线。最终效果保留，候选和成功/失败证据不回滚、不废弃；本轮不继续部署研发。最新优先级与分组分机方向见[权威需求](../requirements/confirmed-platform-scope.md)。

### 当前现场证据与接续边界

PDF、PNG 及正常商品 ZIP 出站已有真实业务证据，不能再以 10 月 11 日“新计划未切换”作为当前部署断点。旧事实保留在[历史摘要](../validation/records/2026-10-11-overall-status-evidence.md)。最后现场观察为 **2026-10-19 北京时间 18:36**：收发关闭、无在途、原 Bot 登录，随后临时授权撤销、临时密码删除；本轮不访问生产，也不推断现在运行状态。

当前正式机器仍使用项目插件和受限客户端；Gateway、FileBrowser 服务端、Windows 客户端、Hermes、WeChat 镜像及报告提交是不同身份，完整值见[本轮固定证据](../validation/records/2026-10-20-overall-status-evidence.md)。以后的部署先核对当时状态和正式发布输入，不重放旧 plan 或重复安装相同工具补证。

### 最终部署标准与候选边界

- 服务端一条正式、可复用的入口配合必要参数完成部署，并保持各组件资源归属、固定源码/制品、权限与恢复边界。
- 电脑只安装官方 Hermes，再配置少量参数即可运行；无需额外项目 EXE、手工复制插件、修改 Hermes 源码或项目专用常驻进程。
- 不依赖 Codex 工作树、`.task-artifacts` 或旧现场回执。
- 临时运维权限撤销后，正式业务凭业务凭据继续运行；管理权限撤销本身不证明业务持续运行已经验收。

新机简部署尚未满足最终标准。PR12 保存的[统一部署与官方 Hermes 最终候选证据](../validation/records/2026-10-20-configuration-only-candidate.md)按语义吸收，准确候选为：

| 候选 | 固定 HEAD | 已有结果与边界 |
| --- | --- | --- |
| Gateway PR24 统一入口 | `b3f2740289c6b1fa3208c352bc87b99b6f50cce4` | 隔离 Debian 13 容器真实统一入口通过，18,168,526 字节/12 成员、失败接续、撤销一次性管理密码后业务凭据可用、同镜像重建与第二次执行保留文件。systemd 观察为替身，tmpfs 非持久宿主盘；新增三组件的 booted-systemd、新机/整机重启仍缺 |
| Gateway PR23 官方 Hermes 原生接入 | `57bb2c2830f292d5167f0bda449afafbe0a528fd` | 官方依赖环境、原生图文与真实 TLS FileBrowser fixture 查询/导出通过；可信当前任务授权和 Artifact/Response/微信替身返回未完成 |
| FileBrowser PR7 | `47876e3a3d62affe777de8623697f62791fa5fa4` | 准备中断恢复、部署资产通过；常规检查的后端 lint 仍有 111 项失败，不写全仓绿 |
| WeChat PR11 | `e56c7e4b606f1614e54210a391906f4be7440f5e` | 组件边界、Debian clean、forced-QR 和 media 检查通过；不含实际微信登录/收发，也未切换协议文件名候选 |

MCP 仍是候选路线，不能写成已迁移或生产已采用。PR23 最终探针使用固定官方 Hermes `427d605d4985c2d49a795493ef6c5d66413057d1`，官方锁定 Python 3.14.7 与默认完整 `[all]` 依赖，53 项适用约束全部满足；旧自选依赖子集和 Windows Python 3.12 诊断不作为正式依赖验收。官方依赖阶段成功不等于完整安装器或 Windows 新机验收，跨机器 MCP HTTPS 尚未证明。

两个业务会话复用一个 MCP transport session，每次调用 `_meta={}`；可信 Gateway task/session/employee 绑定缺失，`trusted_task_context_missing` 实际拒绝且 `task_return_authorized=false`。Profile 字段或静态 header 不提供同 Profile 并发任务的可信身份。尚未进入 Artifact/Response/微信协议替身交付及身份错配、过期、跨会话、重放矩阵。不能让模型填写可信 task ID 或放宽授权补齐结果。两个隔离组合不能拼成已经通过的完整业务链；CI 通过也不代替正式部署/微信实收。准确运行与原失败保留在上述固定记录，本轮没有重跑。

官方 Profile 管理与纯配置文件工具是不同目标：B 方案、Gateway 受控创建/同步 Profile 的方向已确认，但保存引用、实际路由、自动创建、同步更新、已有会话生效时机须分别验证。不能由 MCP 工具发现或官方支持某接口宣称 Gateway 已完成 Profile 管理。

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

本节和第 5、6 节保留**现有运行脚本的旧契约说明**，不是本轮新恢复目标。用户已确认未来恢复入口应先检查：微信在线则保留现有登录，离线再提示扫码，不默认清 runtime、重装或强制 fresh QR。现存 forced-QR、`restart: no` 与 Archive 不复用流程尚未调整，存在明确差异；须另行实现和验收后才能替换，本轮不改脚本、不执行流程。详见[恢复目标与旧契约](../operations/recovery-runbook.md#最新恢复目标与旧运行契约)。

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

### 4.6 旧 forced-QR 契约

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

## 5. 四类变更的旧运行契约

下表是现存契约，不能当作新用户要求“每次恢复必须扫码”；新入口的状态检查和登录保留规则尚未实现/验收。

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

FileBrowser 保持唯一正式文件系统，Gateway/Hermes 通过 API、权限与审计接入；正式原件与工作副本分开。PDF/PNG 和正常 ZIP 出站的限定验收保留；文件名原样保留与 ZIP 入站仍有缺口，通道回传不证明按要求保存后独立检索完成。普通附件默认不处理，按任务取得、明确要求才正式保存，纯文档群可明确配置例外。Skills 在四仓基础完成后由用户找 Hermes 制作并单独建仓；原并行试点安排已调整。路径调整明确暂缓，简部署与恢复入口后置，已有候选按各自真实证据保留。

## 8. 禁止事项

- 把一个项目的 Compose 当成另一个项目的所有者。
- 打印 `.env`、Token、Authorization 或 database URL。
- 把未合并候选或组件文档 PR 写成 main/已部署，或把旧故障手册起点当作最新现场断点。
- 执行适用的旧 forced-QR 入口时擅自绕过其契约；未来恢复入口的在线登录保留目标按本页所列差异另行实现和验收。
- 用手工数据库写入恢复 `uncertain`。
- 把 FileBrowser 限定部署扩大为完整文件业务验收，或把企业 Skills Runtime、独立 OCR、旺店通/S6 的规划写成已经上线。

具体故障流程见[Recovery Runbook](../operations/recovery-runbook.md)。
