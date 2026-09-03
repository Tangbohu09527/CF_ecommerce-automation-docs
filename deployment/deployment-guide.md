# Deployment Guide

> 中文名称：跨项目生产部署指南
>
> 文档编号：DEP-001
>
> 状态日期：2026-09-03

## 1. 适用范围

本文规定企业 Runtime 的跨仓库部署顺序和门禁，不复制组件仓库的完整 Runbook。每次部署必须使用精确 commit、image digest、database revision、Compose project、回滚 Release 和脱敏验证记录。

## 2. 发布输入

| 输入 | 要求 |
| --- | --- |
| 文档 | 本仓库精确 commit |
| Gateway | merged main SHA、production source SHA、immutable image digest、revision `20260823_04` |
| WeChat | 实际批准分支/SHA、immutable image；未完成 main promotion 时明确写 PR 栈 |
| PostgreSQL | image、revision、备份与 restore 计划 |
| Hermes | external runtime 的批准控制和健康检查；不从历史文档猜版本 |
| FileBrowser | 只有形成正式 Candidate 后才加入生产清单 |
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

### 4.3 Gateway core

启动 migration、Gateway API 和 readiness。生产长期应用进程以 `10001:10001` 运行，使用 immutable image 和 Gateway 专属 `64m x 10` 日志策略。

### 4.4 Runtime Controller

核对 `contract`、`status`、ready 和 Token contract。Controller 是 Poll/Delivery Gate 的唯一跨组件控制入口。

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
2. 显式 stop Gate，不假设 Host boot 后已 stopped。
3. 使用组件仓库批准的唯一 forced-QR 入口。
4. 手机扫码。
5. 验证 WeChat 进程、auth、chats、messages 和容器/API health。
6. 失败时保持 Gate 关闭并保留证据。

QR、Cookie、Session、Token、账号和 Chat ID 不进入仓库或普通日志。

### 4.7 Hermes

按批准方式启动或核对 Hermes external runtime，从 CFserver 验证 reachability。不要把 Worker liveness 当成 Hermes connectivity。

### 4.8 Gateway Workers

在前置门禁全部通过后恢复 Poll、Dispatch 和 Delivery Worker。Poll 只入队 durable Dispatch，Dispatch 调用 Hermes，Delivery 只处理持久化 Response/Outbox。

### 4.9 status

必须同时核对：

- database/revision；
- Gateway readiness；
- 三个 Worker heartbeat；
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
| CFserver reboot | 必须 | agent-wechat 保持停止；检查并显式 stop Gate，再 fresh QR |
| Gateway-only deploy | 不需要 | 不重建 agent-wechat，保持 Session |
| agent-wechat restart/recreate | 必须 | stop Gate、Archive、fresh Runtime、QR、API 验证 |
| AI host restart | 通常不需要 | Session 保持；验证 Hermes reachability 与 Queue |

## 6. 升级与回滚

- 先保存当前 SHA、digest、revision、Controller status、日志和 backlog。
- Gateway 回滚到 previous immutable Release；不覆盖 active Release。
- 应用回滚和数据库回滚分开，旧应用必须与当前 schema 明确兼容。
- agent-wechat 代码/镜像变化仍需 stop -> Bootstrap -> fresh QR。
- 不执行 `docker compose down -v`、跨项目 `down`、`--remove-orphans` 或 volume 删除。
- 不删除数据库行、Checkpoint、Queue、Response、Outbox、Archive 或恢复审计。

## 7. FileBrowser future deployment

FileBrowser V1 Beta 已实现和自动化验证，但本指南不把它写入当前实线部署。上线前另行完成：

1. Candidate/immutable image。
2. shared-host ownership 与网络。
3. migration。
4. database/config/business file backup。
5. isolated restore。
6. rollback。
7. real WebDAV and OnlyOffice。
8. File Service permission/audit。
9. Gateway/Hermes integration。

## 8. 禁止事项

- 把一个项目的 Compose 当成另一个项目的所有者。
- 打印 `.env`、Token、Authorization 或 database URL。
- 从未合并组件文档 PR 推断生产命令。
- 用旧 Session 或 Archive 跳过 fresh QR。
- 用手工数据库写入恢复 `uncertain`。
- 把 FileBrowser、Skills、OCR、旺店通或 S6 当作当前生产组件。

具体故障流程见[Recovery Runbook](../operations/recovery-runbook.md)。
