# V2 Enterprise Runtime：CFserver Staging 部署

## 1. 文档信息

| 项目 | 内容 |
| --- | --- |
| 适用版本 | `v2-enterprise-runtime-20260811` |
| 目标环境 | CFserver Staging |
| 操作系统 | Debian 13.6 |
| 文档状态 | 部署准备完成，staging已运行 |
| 配套手册 | [Runtime 运维手册](../operations/runtime-operations.md) |
| 实际状态 | [CFserver Staging 部署状态](./cfserver-staging-status.md) |

本文说明 V2 Enterprise Runtime 在 CFserver 上的部署边界、目录、网络、持久化和上线核对要求。部署准备已经完成，Staging 已运行；截至 2026-08-11，实际运行范围为 `postgres` 与 `gateway`，Hermes API 尚未接入，Workers 和 WeChat runtime 尚未启用。实际版本、镜像、migration 和验证证据统一记录在 [CFserver Staging 部署状态](./cfserver-staging-status.md)。

本文仍不定义端口号、Compose project 名、Secret 值或尚未确认的完整拓扑参数；这些信息必须从对应版本的发布清单和受控环境配置中取得。

## 2. 部署边界

CFserver 是企业 AI 节点，承载 FileBrowser Enterprise、`CF_agent-gateway`、PostgreSQL、Workers 和 Artifact storage 等节点职责。生产 Agent Hermes 仍计划运行在 Windows AI 电脑侧，通过受控链路与 `dispatch-worker` 集成；该链路尚未在本次 Staging 部署中启用。

V2 Enterprise Runtime 的完整目标拓扑包含以下逻辑服务。表中的“当前状态”只描述本次 CFserver Staging 实际运行情况：

| 逻辑服务 | 当前状态 | 部署职责 | 是否需要持久化 | 是否应直接暴露到公网 |
| --- | --- | --- | --- | --- |
| `postgres` | **已运行** | 保存消息归档、线程、时间线、上下文快照、Outbox、响应和投递状态等权威事实 | 是 | 否 |
| `gateway` | **已运行** | 接收内部请求，执行权限检查、事实写入、路由和编排，并提供版本已实现的管理查询能力 | 依据发布配置；核心事实应在 PostgreSQL | 否 |
| `agent-wechat` | **暂未启用** | 对接微信消息通道，将消息交给 Gateway，并承接版本支持的回复投递接口 | 会话或登录数据是否持久化以发布配置为准 | 仅在接入模式确实要求入站端口时开放 |
| `dispatch-worker` | **暂未启用** | 消费 Dispatch Outbox，将已持久化任务交给 Hermes | 否；状态以权威存储为准 | 否 |
| `delivery-worker` | **暂未启用** | 消费已持久化响应并执行投递 | 否；状态以权威存储为准 | 否 |

部署边界必须保持一致：Gateway 负责事实、权限和编排；Hermes 负责推理和工具选择。不得把 Debian 权威状态迁移到 Hermes 本地，也不得让 Hermes 绕过 Gateway 直接修改权威业务事实。

## 3. CFserver 基线

| 项目 | 基线 |
| --- | --- |
| 设备角色 | CFserver，Debian 权威控制中心 |
| 操作系统 | Debian 13.6 |
| CPU | Intel Core i5-12400 |
| 内存 | 16 GB RAM |
| 存储 | RAID1 3.6 TB |
| Docker Engine | 26.1.5 |
| Docker Compose | 2.26.1 |
| 部署配置根目录 | `/opt/cf-agent-gateway` |
| 持久数据根目录 | `/srv/storage/cf-agent-gateway` |

RAID1 只提供磁盘冗余，不等同于备份。数据库备份必须可从独立故障域恢复，且恢复流程需要定期演练。

完整目标拓扑继续启用前至少核对：

- Debian 13 已完成安全更新，系统时间和时区配置正确。
- RAID1 状态正常，文件系统空间和 inode 充足。
- Docker Engine 和实际采用的 Compose 运行方式已安装并锁定兼容版本。
- 主机重启后的容器托管方式已经验证，且不会绕过迁移门禁。
- CFserver 到 Windows Hermes 端点以及必要外部服务的出站策略已明确。
- Staging 使用独立凭据、数据库和微信接入身份，不与生产环境混用。

## 4. 发布输入与上线门禁

以下值不是本文固定配置。部署负责人必须在发布工单或受控配置中记录实际值，并在执行前双人核对：

| 参数 | 值的来源 | 上线要求 |
| --- | --- | --- |
| `<release-package>` | `v2-enterprise-runtime-20260811` 发布产物 | 校验来源、版本和完整性 |
| `<compose-file>` | 发布包 | 通过 `docker compose config` 校验 |
| `<project-name>` | 环境配置 | 与其他环境隔离，不依赖默认目录推导 |
| `<image-reference-or-digest>` | 发布清单或私有镜像仓库 | 建议锁定不可变 digest，不使用漂移的 `latest` |
| `<postgres-service>` 等服务标识 | Compose 清单 | 与本文逻辑服务逐一映射 |
| `<migration-command>` | 本版本发布说明 | 必须明确、可审计，禁止猜测 |
| `<gateway-health-url>` | Gateway 本版本配置 | 明确绑定地址、端口和健康路径 |
| `<worker-heartbeat-source>` | Runtime 本版本配置 | 明确查询入口和告警阈值 |
| `<hermes-endpoint>` | 受控环境配置 | 验证网络、认证和超时策略 |
| Secret 与微信登录数据 | Secret 管理流程 | 不写入 Git、镜像、文档或命令历史 |

任一关键项仍为占位符时不得执行上线。生产运行不得依赖 GitHub 持续在线；发布产物和所需镜像应在维护窗口前完成受控准备。

## 5. 目录与持久化规划

### 5.1 目录职责

| 路径 | 用途 | 要求 |
| --- | --- | --- |
| `/opt/cf-agent-gateway` | 发布清单、Compose 配置、经过审核的非敏感环境配置及运维入口 | 发布版本可追溯；普通运行数据不得混入 |
| `/srv/storage/cf-agent-gateway` | PostgreSQL 数据及发布配置明确要求的持久运行数据 | 独立权限、容量监控、备份和恢复验证 |

可在这两个根目录下按环境、版本和数据类型划分子目录，但具体目录名和容器挂载点必须服从发布包，不能只凭本文创建。升级时不得用新发布包覆盖 PostgreSQL 数据目录。

### 5.2 Volume 规划

| 数据类型 | 推荐承载方式 | 说明 |
| --- | --- | --- |
| PostgreSQL data | Docker named volume 或 `/srv/storage/cf-agent-gateway` 下的 bind mount，二选一并固定 | 宿主路径与容器内数据目录以 PostgreSQL 镜像和发布清单为准 |
| Gateway/Worker 文件状态 | 仅在发布配置明确要求时挂载 | 不把数据库权威状态复制成无审计的本地文件状态 |
| `agent-wechat` 会话或登录状态 | 仅按适配器验证结果和发布配置持久化 | 按 Secret 等级保护，不得提交到 Git 或进入普通备份分发范围 |
| 文件日志 | 仅在服务不使用标准输出时挂载 | 默认优先由 Docker 日志驱动统一收集并配置轮转 |
| 数据库备份 | 独立备份目标 | 不得仅放在同一 RAID1 阵列并视为有效备份 |

挂载前确认宿主目录 owner、group 和 mode 与容器运行身份匹配。不得为解决权限问题使用全局可写权限，也不得在不理解 UID/GID 映射时递归修改整个存储根目录。

## 6. Docker Network 与端口规划

### 6.1 网络分区

建议由发布 Compose 清单创建一个用户定义的内部 bridge network，本文以 `<backend-network>` 表示其逻辑角色，实际名称不作固定。完整目标拓扑中的五个服务通过该网络使用容器 DNS 通信；当前仅确认 `postgres` 与 `gateway` 已运行。

| 通信方向 | 允许条件 |
| --- | --- |
| `agent-wechat` -> `gateway` | 仅访问 Gateway 内部监听地址；必须携带版本要求的认证信息 |
| `gateway`/workers -> `postgres` | 仅容器网络内访问；数据库不映射宿主公网端口 |
| `dispatch-worker` -> Hermes | 只允许到 `<hermes-endpoint>` 的必要出站连接，认证材料由 Secret 提供 |
| `delivery-worker` -> `agent-wechat` 或实际通道端点 | 以发布版投递链路为准，只开放必要出站连接 |
| 管理员 -> `gateway` Admin API | 仅管理网或 loopback，经 admin role 鉴权 |

当前 Hermes remote transport 尚未独立抽象。部署配置必须按本版本已经实现的 Hermes Integration 方式核对，不能把未来 transport 设计当作可用配置。

### 6.2 端口策略

| 服务 | 容器内端口 | 宿主机映射策略 |
| --- | --- | --- |
| `postgres` | 由所用镜像和发布清单确定 | 默认不映射；确需临时维护时只绑定 loopback，并在维护后撤销 |
| `gateway` | `<gateway-container-port>` | 仅在宿主健康检查、反向代理或受控管理访问需要时绑定 `<gateway-host-port>`；优先 loopback 或管理网地址 |
| `agent-wechat` | `<agent-wechat-container-port>`，仅当接入模式需要 | 仅开放适配器实际要求的端口；公网回调必须经过已批准的 TLS、鉴权和防火墙策略 |
| `dispatch-worker` | 无入站端口需求 | 不映射 |
| `delivery-worker` | 无入站端口需求 | 不映射 |

端口号、健康路径和监听地址必须在上线前从实际配置解析。不得将示例占位符替换成惯例端口后直接上线，也不得在无访问控制时绑定 `0.0.0.0`。

## 7. 配置与 Secret

- 环境配置必须标注 `staging`，并使用与生产隔离的数据库、Hermes 凭据和通道凭据。
- Secret 不得出现在 Compose 文件、镜像层、Git 历史、文档、截图或工单明文中。
- 数据库账号按服务最小权限分配；迁移账号与日常运行账号能否分离，以本版本实现和发布清单为准。
- Admin API 仅允许 `admin role`，入口限制到管理网或 loopback，并保留审计。
- 日志不得输出消息全文、Cookie、Token、微信登录数据或数据库连接密码。
- 配置变更需记录变更人、时间、版本和回退值。不得在容器内直接修改配置形成不可追溯漂移。

## 8. 部署流程

### 8.1 准备与校验

1. 在 `/opt/cf-agent-gateway` 下准备与 `v2-enterprise-runtime-20260811` 对应的发布产物，不从未固定的分支直接部署。
2. 核对镜像引用、实际 Compose service 标识、网络、volume、端口、健康检查、迁移入口和 Secret 注入方式。
3. 对现有数据库执行可恢复备份，并记录备份标识和恢复校验结果。
4. 确认 `/srv/storage/cf-agent-gateway` 的目标挂载、权限、剩余容量和备份策略。
5. 生成最终 Compose 配置并人工复核，不将渲染后的 Secret 输出保存到不受控位置。

以下命令仅展示参数化校验形式，不是可直接执行的发布命令：

```console
docker compose -f <compose-file> -p <project-name> config
```

占位符必须替换为发布工单中的已审核值。在 Debian shell 中执行时同样不得把 Secret 直接拼入命令行。

### 8.2 完整拓扑启动顺序

完整拓扑按以下顺序执行，详细检查点见 [Runtime 运维手册](../operations/runtime-operations.md)。当前部署流程已经执行至 Gateway 启动；已验证范围以第 10.1 节为准，第 4、5 步仍是计划：

1. 启动 database，即实际映射到 `postgres` 的服务，并等待 readiness 通过。
2. 执行本版本 migration，确认成功后才能继续。
3. 启动 `gateway`，确认容器和应用健康检查均通过。
4. 启动 `dispatch-worker` 与 `delivery-worker`，确认 heartbeat 正常。
5. 启动或放通 `agent-wechat`，在端到端链路健康后再接收真实流量。

下面只表示执行形状，实际命令、service 名和 migration 入口以发布包为准：

```text
docker compose -f <compose-file> -p <project-name> up -d <postgres-service>
<migration-command-from-release-package>
docker compose -f <compose-file> -p <project-name> up -d <gateway-service>
docker compose -f <compose-file> -p <project-name> up -d <dispatch-worker-service> <delivery-worker-service>
docker compose -f <compose-file> -p <project-name> up -d <agent-wechat-service>
```

禁止跳过 migration，禁止在 migration 失败后带错继续启动，也禁止把 migration 作为每次容器重启时都会无条件执行的普通启动步骤。

## 9. Systemd 与开机托管

Docker/Systemd 是本版本支持的部署方式，但本文不虚构 unit 名或 unit 内容。部署时遵循以下规则：

- Docker daemon 由 systemd 管理并设置为按主机策略启动。
- Compose 服务使用发布包确认的 restart policy，或使用经过审核的 systemd unit 托管；同一职责只保留一种启动所有权。
- 若发布包提供 systemd unit，必须核对其版本、工作目录、Compose 文件、project 名和依赖关系后安装。
- 若发布包未提供 unit，应先补充独立评审的部署资产，不能从本文臆造一个 unit 投入运行。
- systemd 启动依赖至少应覆盖 Docker 和必要网络就绪条件；停止时应先停止入口和 workers，再停止 Gateway，最后才允许停止数据库。
- migration 作为受控发布步骤执行，不应仅依赖通用 `ExecStartPre` 在每次重启时盲目运行。
- unit、EnvironmentFile 和 journal 中不得泄露 Secret。

当前运行服务的主机重启演练必须验证：服务不会抢在 migration 门禁前处理流量，且数据库 volume 未变化。Workers 启用后，还必须验证 worker heartbeat 恢复和积压事实可继续处理。

## 10. 验证状态

### 10.1 当前已验证

- `postgres:16` 与 `cf-agent-gateway:v2-enterprise-runtime-20260811` 已启动。
- 数据目录挂载正常。
- Alembic migration 已到 `20260810_01 (head)`。
- Gateway `GET /ready` 返回 `{"status":"ready"}`。

完整记录见 [CFserver Staging 部署状态](./cfserver-staging-status.md)。

### 10.2 完整拓扑基础检查（待完成）

- 所有容器的实际镜像版本与发布清单一致。
- `docker ps`/Compose 状态无反复重启，健康状态满足发布标准。
- PostgreSQL readiness 正常，目标 schema migration 已成功完成。
- Gateway 健康检查通过，Admin API 不能被非 admin 身份访问。
- 两个 worker 的 heartbeat 在规定窗口内持续更新。
- CFserver 到 Hermes 的网络和认证验证通过。
- Docker network 未包含无关容器，PostgreSQL 未暴露到公网。

### 10.3 端到端检查（待完成）

使用脱敏 Staging 测试消息验证以下事实链，禁止使用真实业务文件或真实 Secret：

1. Message Archive 出现唯一且可追溯的入站事实。
2. Routing Runtime 和 ThreadResolver 产生预期线程归属。
3. Dispatch Outbox 记录进入可处理状态，`dispatch-worker` 将任务交给 Hermes。
4. Context Runtime 从已持久事实构建当前上下文；Snapshot 只作为缓存，不替代 Timeline。
5. Hermes 响应已经持久化后，`delivery-worker` 执行投递。
6. 投递结果可通过当前 Admin Archive API 的只读能力和日志交叉核对。

验证结论应记录关联 ID、时间、版本和结果，不在报告中复制敏感消息正文。

## 11. 回退原则

- 应用回退前先停止 `agent-wechat` 入站和 workers，保护现有 Message Archive、Outbox、响应与投递事实。
- 只有在旧应用版本与当前数据库 schema 明确兼容时，才允许回退应用镜像。
- migration 回退必须使用本版本提供并验证过的方案；不得手工删表、改版本号或恢复单张表来伪造成功状态。
- 需要恢复数据库时，使用部署前可验证备份并完整记录恢复点。RAID1 不能代替恢复点。
- 回退后仍按 database -> migration 状态核对 -> gateway -> workers -> `agent-wechat` 放流顺序恢复。

## 12. 相关文档

- [项目总纲](../../00_项目总纲.md)
- [部署运维总则](../../04_部署运维.md)
- [V2 Enterprise Runtime 架构总览](../architecture/v2-enterprise-runtime.md)
- [CFserver Staging 部署状态](./cfserver-staging-status.md)
- [Runtime 运维手册](../operations/runtime-operations.md)
- [当前限制](../status/current-limitations.md)
