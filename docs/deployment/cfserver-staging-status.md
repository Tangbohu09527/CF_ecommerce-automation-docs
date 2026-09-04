# CFserver V2 Enterprise Runtime Staging 部署状态

> **历史快照警告**
>
> 本文是 **2026-08-11 CFserver Staging 历史快照**，仅用于版本追溯。其服务状态、拓扑、参数和命令可能已被后续生产基线取代，**禁止直接作为当前生产部署或运维操作依据**。
>
> 当前操作请以[生产部署指南](../../deployment/deployment-guide.md)、[故障恢复手册](../../operations/recovery-runbook.md)和[当前状态矩阵](../../status/current-status.md)为准。
> 当前生产证据见[2026-09-03 Production Closeout](../../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)。

> 状态日期：2026-08-11
> 适用版本：`v2-enterprise-runtime-20260811`
> 文档状态：实际部署记录

本文只记录已经在 `CFserver` Staging 环境确认的事实。未列入“已完成”或“已验证”的能力仍属于计划，不因代码已发布或服务已出现在目标拓扑中而视为可用。

## 1. CFserver 角色说明

`CFserver` 是企业 AI 节点，也是 Debian 权威控制中心的实际承载主机。分配给该节点的职责包括：

- FileBrowser Enterprise
- `CF_agent-gateway`
- PostgreSQL
- Workers
- Artifact storage

上述列表描述节点职责，不代表所有组件均已启用。截至 2026-08-11，本次 V2 Enterprise Runtime Staging 部署只确认 PostgreSQL 与 Gateway 正在运行；FileBrowser Enterprise 的运行状态不在本次部署记录的验证范围内，Workers、Hermes 和微信运行时的状态见下文。

### 环境基线

| 项目 | 实际值 |
| --- | --- |
| 服务器 | `CFserver` |
| 操作系统 | Debian 13.6 |
| CPU | Intel Core i5-12400 |
| 内存 | 16 GB RAM |
| 存储 | RAID1 3.6 TB |
| Docker Engine | 26.1.5 |
| Docker Compose | 2.26.1 |

RAID1 提供磁盘冗余，不等同于独立备份。

### 部署版本

| 项目 | 实际值 |
| --- | --- |
| 代码版本 / Git tag | `v2-enterprise-runtime-20260811` |
| Commit | `2ac4c86dbcbb3ac035c3688100e88c57407575b7` |
| PostgreSQL 镜像 | `postgres:16` |
| Gateway 镜像 | `cf-agent-gateway:v2-enterprise-runtime-20260811` |

## 2. 当前 Docker 服务状态

| 服务 | 当前状态 | 说明 |
| --- | --- | --- |
| `postgres` | **已运行** | 数据目录挂载正常，migration 已成功执行 |
| `gateway` | **已运行** | Docker 启动正常，Gateway readiness 已验证 |
| `dispatch-worker` | **暂未启用** | 等待 Hermes API 接入 |
| `delivery-worker` | **暂未启用** | 等待 WeChat runtime 接入 |

因此，当前状态是 Gateway 与数据库基础运行环境已经可用，不代表消息分发、Hermes 推理、微信投递或完整端到端业务链路已经跑通。

## 3. 目录结构

部署目录与数据目录分离：

```text
/opt/cf-agent-gateway/
└── 代码和部署文件

/srv/storage/cf-agent-gateway/
├── postgres/
├── artifacts/
├── context/
├── uploads/
├── logs/
└── backups/
```

- `/opt/cf-agent-gateway/` 保存代码和部署文件。
- `/srv/storage/cf-agent-gateway/` 保存 PostgreSQL、产物、上下文、上传、日志和备份数据。
- 本次已确认数据目录挂载正常；备份目录存在不等于备份恢复已经完成演练。
- 这些运行时目录不改变正式文件访问边界：正式文件仍必须经过 File Service、权限检查和审计，`context/` 也不替代 PostgreSQL 中的权威事实。

## 4. 部署流程

本次 Staging 部署按以下顺序完成：

```text
Git tag
  ↓
Debian clone
  ↓
docker build
  ↓
Docker Compose 启动 PostgreSQL
  ↓
Alembic migration
  ↓
Gateway 启动
```

部署使用固定版本和 commit，不依赖未固定分支作为运行基线。Workers 与微信运行时不在本次已完成启动范围内。

## 5. 当前验证结果

### 已完成

- Docker 启动正常。
- 数据目录挂载正常。
- Alembic migration 成功。
- Gateway health 正常。

### Gateway readiness

请求：

```http
GET /ready
```

返回：

```json
{"status":"ready"}
```

### 数据库 migration

Alembic 当前版本：

```text
20260810_01 (head)
```

已执行的 migration 链：

```text
20260806_01
  ↓
20260806_0001
  ↓
20260806_0002
  ↓
20260806_02
  ↓
20260806_03
  ↓
20260806_04
  ↓
20260807_01
  ↓
20260807_02
  ↓
20260807_03
  ↓
20260810_01
```

## 6. 下一阶段计划

以下项目尚未完成：

- 接入 Hermes API。
- 启动并验证 `dispatch-worker`。
- 部署 `agent-wechat`。
- 在微信运行时可用后启动并验证 `delivery-worker`。
- 完成微信真实联调。
- 验证 Context Runtime。

完整部署要求见 [Staging Debian 部署基线](./staging-debian.md)，当前能力限制见 [V2 当前限制](../status/current-limitations.md)，启停与排障步骤见 [Runtime 运维手册](../operations/runtime-operations.md)。
