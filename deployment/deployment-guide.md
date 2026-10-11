# Deployment Guide

> 中文名称：跨项目生产部署指南
>
> 文档编号：DEP-001
>
> 状态同步日期：2026-10-11。以下为通用部署边界；本次只同步文档，不执行部署或恢复。

## 1. 适用范围

本文规定企业 Runtime 的跨仓库部署顺序和门禁，不复制组件仓库的完整 Runbook。每次部署必须使用精确 commit、image digest、database revision、Compose project、回滚 Release 和脱敏验证记录。

### 2026-10-11 当前接续点

用户回传：旧 f9 两端回退、d9 `complete-rollback` 与旧核心恢复已完成；随后 d9 新候选镜像与两端新 plan 通过，尚未切换，新 deployment 未 `apply/start/verify`，微信附件实收未验收。用户之后只重启了 CFserver；重启后状态没有核验，Windows 是否关机未确认。

下一步先核验重启后现场与现存新计划，确认其身份、阶段、回退基线和当前环境仍匹配后再接续切换。不得从历史故障手册的起点重新回退、传包、构建或为补证重放 apply；不得沿用重启前健康和历史 PID。候选、源码、PR/CI 与现场事实集中在[10 月 11 日证据摘要](../validation/records/2026-10-11-overall-status-evidence.md)。本页通用流程不表示本次已经恢复或取得执行授权。

## 2. 发布输入

| 输入 | 要求 |
| --- | --- |
| 文档 | 本仓库精确 commit |
| Gateway | 当前 main、获准候选/source SHA、合并状态、实际 Image ID、可独立核实的 registry digest 与目标 schema 分列；本次 d9 修复线尚未合并，不能写成 main |
| WeChat | 实际批准分支/SHA、immutable image；未完成 main promotion 时明确写 PR 栈 |
| PostgreSQL | image、revision、备份与 restore 计划 |
| Hermes | external runtime 的批准控制和健康检查；不从历史文档猜版本 |
| FileBrowser | 已有同机部署和受限文件操作证据；继续核对实际 image、候选合并状态、权限与恢复范围，不能由旧待部署条目否定已有现场进展 |
| Secret | 只写受控引用，不记录值、hash/prefix 或完整环境文件 |

未解析占位符、漂移 `main`、`latest` 镜像或缺失回滚目标都会阻断部署。

## 3. 基础环境

- CFserver Host 使用 `Asia/Shanghai`，容器和 PostgreSQL 使用 UTC。
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

先核对目标 schema 与当前 Release 的兼容性，仅在获准的迁移窗口执行需要的 migration；普通重启和本次计划接续不默认运行 upgrade。Gateway API 与 Dispatch Worker 分别验证 readiness/heartbeat。Dispatch Worker 由 Gateway Release/Compose 生命周期负责，不属于 Runtime Controller 的 controlled services；组合 Poll/Delivery Gate 关闭时，Gateway API 与 Dispatch Worker 可以保持在线。生产长期应用进程以 `10001:10001` 运行，使用 immutable image 和 Gateway 专属 `64m x 10` 日志策略。

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

09-25 至 09-27 用户回传已有 FileBrowser 同机部署、HTTPS 受限接入、微信真实文件读取及一次受控文本新建证据。10-07 Gateway 固定文档补充两个指定 PDF/PNG 的文件 API 下载、内容读取与原聊天文字结果关联，仍不证明微信入站附件取件或原聊天附件实收。各能力范围见[状态矩阵](../status/current-status.md)及[10 月 11 日摘要](../validation/records/2026-10-11-overall-status-evidence.md)。

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

FileBrowser 保持唯一正式文件系统，Gateway/Hermes 通过 API、权限与审计接入；正式原件与工作副本分开。当前先完成微信真实文件/图片收发，分类归档和加工回存沿用[既有业务需求](../requirements/file-workflows.md)，不并行扩安装器。

## 8. 禁止事项

- 把一个项目的 Compose 当成另一个项目的所有者。
- 打印 `.env`、Token、Authorization 或 database URL。
- 把未合并候选或组件文档 PR 写成 main/已部署，或把旧故障手册起点当作最新现场断点。
- 用旧 Session 或 Archive 跳过 fresh QR。
- 用手工数据库写入恢复 `uncertain`。
- 把 FileBrowser 限定部署扩大为完整文件业务验收，或把企业 Skills Runtime、独立 OCR、旺店通/S6 的规划写成已经上线。

具体故障流程见[Recovery Runbook](../operations/recovery-runbook.md)。
