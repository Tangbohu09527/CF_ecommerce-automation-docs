# V2 Enterprise Runtime 运维手册

> **历史快照警告**
>
> 本文是 **2026-08-11 CFserver Staging 历史快照**，仅用于版本追溯。其服务状态、拓扑、参数和命令可能已被后续生产基线取代，**禁止直接作为当前生产部署或运维操作依据**。
>
> 当前操作请以[生产部署指南](../../deployment/deployment-guide.md)、[故障恢复手册](../../operations/recovery-runbook.md)和[当前状态矩阵](../../status/current-status.md)为准。

## 1. 文档信息

| 项目 | 内容 |
| --- | --- |
| 适用版本 | `v2-enterprise-runtime-20260811` |
| 适用环境 | CFserver Staging；其他环境需使用各自受控参数 |
| 目标 | 让 Runtime 可启动、可检查、可止损、可恢复 |
| 部署基线 | [CFserver Staging 部署](../deployment/staging-debian.md) |

本手册只覆盖当前发布版已经具备的 Message Archive、V2 Routing Runtime、ThreadResolver、Dispatch Outbox、Dispatch Worker、Hermes Integration、Context Runtime、Context Snapshot、Response Persistence、Delivery Worker 和 Admin Archive API。Skill Runtime、Memory Runtime、RAG、embedding、vector database、automatic memory、WDT/S6 均不得作为当前运维依赖。

## 2. 运维原则

1. **事实优先。** Message Archive、Timeline、Outbox、Response Persistence 和 Delivery 状态是故障恢复依据，不得为了“恢复运行”直接删除或覆盖事实记录。
2. **迁移先行。** database readiness 通过后必须先完成当前版本 migration，才能启动 Gateway 和 workers。不得跳过、假成功或在失败后继续放流。
3. **先持久化，后执行。** 排查 Dispatch 或 Delivery 时，先确认相关事实已经持久化，再判断执行侧故障。
4. **最小化变更。** 一次只改变一个故障变量，记录命令、时间、服务、版本和结果。
5. **只读诊断。** 优先使用 `docker ps`、日志、health、heartbeat 和 Admin API 的只读查询。没有经发布流程确认，不直接更新数据库状态。
6. **保护敏感数据。** 日志和故障记录不得包含 Cookie、Token、微信登录数据、数据库密码或真实消息全文。

## 3. 运行参数登记

本手册使用占位符，值必须来自当前发布清单和受控环境配置。值未确认时停止操作，不按惯例猜测。

| 占位符 | 含义 |
| --- | --- |
| `<compose-file>` | 本版本实际 Compose 文件 |
| `<project-name>` | 当前环境 Compose project 名 |
| `<postgres-service>` | 映射到 `postgres` 的实际 service 名 |
| `<gateway-service>` | 映射到 `gateway` 的实际 service 名 |
| `<dispatch-worker-service>` | Dispatch Worker 实际 service 名 |
| `<delivery-worker-service>` | Delivery Worker 实际 service 名 |
| `<agent-wechat-service>` | `agent-wechat` 实际 service 名 |
| `<migration-command>` | 发布包确认的 migration 命令 |
| `<gateway-health-url>` | 已批准的 Gateway health URL |
| `<container-name>` | `docker inspect` 使用的实际容器名或 ID |
| `<worker-heartbeat-source>` | 本版本确认的 heartbeat 查询入口 |
| `<duration>` | 日志回看窗口，例如经现场决定的分钟数或小时数 |

以下 Compose 示例都是命令形状，不是已固定的发布命令。不得原样执行含尖括号的命令。

## 4. 标准启动

### 4.1 启动前检查

- 当前部署产物明确对应 `v2-enterprise-runtime-20260811`。
- 数据库、Gateway、workers 和 `agent-wechat` 的镜像引用与发布清单一致。
- `/srv/storage/cf-agent-gateway` 挂载、权限、容量和 RAID1 状态正常。
- 最近一次可恢复备份可识别；涉及 migration 时已创建并验证维护窗口备份。
- Compose 渲染结果、Docker network、volume、端口和 Secret 注入已复核。
- 上一次异常停止或 migration 失败已有结论，不存在未处理的半迁移状态。
- `agent-wechat` 尚未对真实流量放通。

配置校验形式：

```console
docker compose -f <compose-file> -p <project-name> config
```

注意：渲染结果可能包含敏感值，不得贴入群聊、工单或仓库。

### 4.2 启动顺序

#### 1. Database

启动 PostgreSQL，先确认容器处于运行状态，再确认数据库 readiness。仅看到容器 `running` 不等于数据库可连接。

```text
docker compose -f <compose-file> -p <project-name> up -d <postgres-service>
docker compose -f <compose-file> -p <project-name> ps <postgres-service>
```

若发布清单配置了 Docker healthcheck，应等待 `healthy`。若本版本规定使用 `pg_isready` 或其他数据库检查，必须采用发布文档给出的用户、数据库和执行方式，不在本手册虚构参数。

#### 2. Migration

在 Gateway 和 workers 保持停止、`agent-wechat` 不接收流量的情况下执行：

```text
<migration-command-from-release-package>
```

继续启动前必须同时满足：

- 迁移进程退出成功。
- 迁移日志无被忽略的 error 或 partial failure。
- 当前 schema 版本与 `v2-enterprise-runtime-20260811` 发布要求一致。
- 迁移执行记录、操作者、开始/结束时间和日志位置已登记。

任何一项不满足都按“Migration 异常”处理，禁止跳过 migration。

#### 3. Gateway

```text
docker compose -f <compose-file> -p <project-name> up -d <gateway-service>
docker compose -f <compose-file> -p <project-name> ps <gateway-service>
```

确认容器稳定运行，并使用 `<gateway-health-url>` 验证应用 health。还应核对数据库连接、schema 版本和必要依赖；只有 HTTP 进程可访问但数据库不可用，不能判定健康。

#### 4. Workers

先后启动 Dispatch Worker 和 Delivery Worker；启动后分别确认容器状态、日志和 heartbeat。

```text
docker compose -f <compose-file> -p <project-name> up -d <dispatch-worker-service> <delivery-worker-service>
docker compose -f <compose-file> -p <project-name> ps <dispatch-worker-service> <delivery-worker-service>
```

workers 健康后再启动 `agent-wechat`。完成一条脱敏端到端验证且投递状态可查询后，才允许接入真实流量。

## 5. 常规检查

### 5.1 `docker ps`

```console
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
docker compose -f <compose-file> -p <project-name> ps
```

检查重点：

- 五个逻辑服务是否都映射到预期容器。
- Image 是否与发布清单一致。
- Status 是否出现 `Restarting`、`Exited` 或长期 `starting`。
- PostgreSQL 是否意外映射了宿主端口。
- Gateway 或 `agent-wechat` 是否暴露了未经批准的地址和端口。

### 5.2 Logs

```text
docker compose -f <compose-file> -p <project-name> logs --since <duration> <service>
docker compose -f <compose-file> -p <project-name> logs --tail <line-count> <service>
```

按时间和关联 ID 串联日志，优先检查：

| 服务 | 重点信号 |
| --- | --- |
| `postgres` | readiness、连接数、磁盘、WAL、恢复或崩溃循环 |
| `gateway` | 请求状态、权限拒绝、数据库错误、归档/路由/线程解析错误 |
| `dispatch-worker` | heartbeat、Outbox 消费、Hermes 连接、超时和重试结果 |
| `delivery-worker` | heartbeat、响应读取、通道调用、投递错误和状态持久化 |
| `agent-wechat` | 通道连接、认证、入站/出站关联 ID；不得泄露登录材料 |

日志出现消息正文或 Secret 时，不应继续转发日志；先按安全事件流程收敛访问范围，再修正日志配置。

### 5.3 Health

若镜像定义了 healthcheck，可查看：

```console
docker inspect --format "{{json .State.Health}}" <container-name>
```

Gateway 应使用发布配置中的真实 health URL：

```text
curl --fail --silent --show-error <gateway-health-url>
```

判定时区分三层：

1. **Container health**：进程是否在运行，是否通过镜像 healthcheck。
2. **Application health**：Gateway 是否能完成版本规定的依赖检查。
3. **Pipeline health**：消息归档、Dispatch、Hermes、响应持久化和 Delivery 是否可完成。

单层通过不能替代完整链路健康。

### 5.4 Heartbeat

对 `dispatch-worker` 和 `delivery-worker` 分别核对：

- worker identity 与环境是否匹配。
- `last seen` 是否在发布配置规定的窗口内。
- heartbeat 是否连续，而不是只在启动瞬间出现一次。
- 运行版本是否为 `v2-enterprise-runtime-20260811`。
- heartbeat 正常时，实际队列是否仍持续积压。

heartbeat 的实际来源可能是当前实现提供的只读管理查询、权威数据库只读查询或结构化日志，以 `<worker-heartbeat-source>` 登记的方式为准。本文不假定 endpoint、表名或字段名；不得为检查 heartbeat 直接写数据库。

### 5.5 每次巡检记录

至少记录以下项目：

| 项目 | 记录内容 |
| --- | --- |
| 时间 | 使用带时区的时间戳 |
| 版本 | 发布 tag 和实际镜像引用/digest |
| 容器 | 状态、restart count、health |
| Database | readiness、容量、备份状态 |
| Workers | heartbeat、最后成功处理时间、积压变化 |
| Delivery | 成功/失败趋势和最早未完成记录时间 |
| 变更 | 自上次巡检后的部署或配置变更 |

不在巡检记录中复制消息正文、文件内容或 Secret。

## 6. 故障处理

### 6.1 Worker 停止或 heartbeat 过期

**现象**

- 容器为 `Exited`/`Restarting`。
- 容器仍在运行，但 heartbeat 超过规定窗口。
- Dispatch 或 Delivery 积压持续增长。

**处置**

1. 记录故障时间、服务、容器 ID、镜像版本、最后 heartbeat 和关联错误，不先删除容器或数据。
2. 用 `docker ps`/Compose `ps` 区分容器停止、health 异常和进程存活但逻辑卡住。
3. 查看故障前后的 worker 日志，并同时检查 PostgreSQL、Gateway、Hermes 或投递通道依赖。
4. 通过只读查询确认待处理事实仍在 Outbox、Response Persistence 或 Delivery 状态中。不得删除积压或手工改成成功。
5. 原因明确且依赖恢复后，只重启受影响 worker，一次一个；使用发布方式重启，不在容器内临时改代码或配置。
6. 验证新 heartbeat、积压下降、同一关联 ID 没有产生异常重复，再解除告警。

如果进程反复崩溃，停止自动重启循环并保留日志。只有发布版明确支持相应重试操作时才能触发重试，不能通过复制消息或直接改数据库制造新的处理任务。

### 6.2 Database 异常

**现象**

- readiness 失败或连接大量超时。
- PostgreSQL 容器反复重启。
- 磁盘满、文件系统只读、RAID1 降级或数据目录权限异常。

**处置**

1. 立即停止 `agent-wechat` 放流，防止持续接收无法可靠归档的新消息。
2. 停止 workers 的继续消费；Gateway 若无法保证事实写入，应从入口摘除或停止。
3. 记录错误和宿主状态，检查空间、inode、内存、RAID1、文件系统、volume 挂载和 PostgreSQL 日志。
4. 不初始化新数据库、不删除 volume、不清理 WAL、不递归修改数据目录权限。
5. 对基础设施问题先恢复底层稳定；涉及数据损坏时使用已批准的 PostgreSQL 恢复流程和已验证备份。
6. 数据库恢复后重新验证 readiness 和数据一致性，再按 database -> migration 状态核对 -> gateway -> workers -> `agent-wechat` 放流的顺序恢复。
7. 核对故障窗口内的 Message Archive、Outbox、Response 和 Delivery 事实，确认没有通过旁路丢失或伪造状态。

RAID1 降级即使数据库仍可运行，也应作为存储故障处理；RAID1 不能证明备份可恢复。

### 6.3 Delivery 失败

**现象**

- Hermes 响应已生成，但用户未收到。
- Delivery 状态持续失败或长时间不推进。
- `delivery-worker` heartbeat 正常但积压增长。

**处置**

1. 按关联 ID 确认入站消息、线程、Dispatch 和 Hermes 响应，尤其确认 Response Persistence 已成功。
2. 确认失败发生在响应持久化之前还是之后。响应未持久化的问题不能按单纯 Delivery 故障重试。
3. 检查 `delivery-worker` 日志、heartbeat、积压和数据库连接。
4. 检查 `agent-wechat` 或实际投递端点的连接、会话、认证、权限、限流和出站网络。
5. 区分单条数据问题与通道级故障；通道级故障时暂停放流或控制新增积压，保留所有已持久化状态。
6. 只使用当前版本正式提供的重试语义。重试前核对重复投递风险，重试后核对投递事实和通道结果。
7. 不得直接把失败记录改为成功，不得删除失败记录来让监控恢复绿色。

恢复标准：失败原因已消除，worker heartbeat 正常，历史积压稳定下降，新消息可以完成 Response Persistence -> Delivery，且未发现异常重复投递。

### 6.4 Migration 异常

**现象**

- migration 非零退出、超时或连接中断。
- 日志出现 partial apply、锁等待或 schema 不匹配。
- Gateway 报告 schema 不兼容。

**处置**

1. 保持 Gateway、workers 和 `agent-wechat` 停止，不放流。
2. 保存完整 migration 输出、数据库日志、开始/结束时间、发布版本和操作者信息。
3. 使用版本支持的只读方式确认 migration 当前状态，区分“未开始”“完整失败”“部分应用”和“已成功但检查失败”。
4. 检查数据库 readiness、权限、锁、空间和版本兼容性。
5. 不盲目重复执行，不跳过失败步骤，不手工修改 migration 版本表，不直接执行未经发布验证的 SQL。
6. 按本版本 migration 说明决定继续、修复后重跑或从部署前备份恢复；没有明确方案时升级给发布负责人。
7. 只有 migration 成功且 schema 状态与发布要求一致后，才可继续启动 Gateway。

应用镜像回退不代表数据库自动回退。只有发布说明明确旧版本与当前 schema 兼容时，才允许回退应用；否则保持停机并保护现场。

## 7. 安全停止与恢复

计划停止时采用以下顺序：

1. 停止 `agent-wechat` 接收新流量或从入口摘除。
2. 等待或受控停止 `dispatch-worker` 与 `delivery-worker`，记录剩余积压。
3. 停止 Gateway。
4. 确认数据库写入和备份状态后，最后停止 PostgreSQL。

恢复时使用相反的依赖顺序，但 migration 始终是 database readiness 之后、Gateway 之前的独立门禁。任何异常停机后都应核对 Archive、Outbox、Response 和 Delivery 事实，而不是仅确认容器重新变绿。

## 8. 升级与维护窗口

- 先记录当前镜像、配置、schema、volume 和可恢复备份，再部署新版本。
- 暂停真实流量并记录积压边界，避免无法判断升级前后责任归属。
- migration 只执行发布包规定的命令，并保留完整执行记录。
- 一次只升级一个发布版本路径；不能把未发布功能配置混入当前版本。
- 完成容器、health、heartbeat 和脱敏端到端验证后再放流。
- 维护结束后检查数据库备份、Docker 日志轮转、磁盘空间和 RAID1 状态。

## 9. 故障升级信息

需要升级给发布负责人或开发负责人时，至少提供：

- 环境、发布 tag、实际镜像引用或 digest。
- 故障开始时间、时区、影响范围和当前是否已停止放流。
- 相关服务状态、health、heartbeat 和 restart count。
- 已脱敏日志片段及关联 ID，不提供消息正文和 Secret。
- 数据库 readiness、存储容量、RAID1 和最近备份状态。
- 已执行操作及每步结果。
- Archive、Outbox、Response、Delivery 事实是否仍可只读查询。

## 10. 恢复完成标准

只有同时满足以下条件，故障才可关闭：

- 根因或明确的触发条件已记录，不只以“重启后恢复”结案。
- 服务镜像、配置和数据库 schema 与发布要求一致。
- Database readiness、Gateway health、两个 worker heartbeat 均正常。
- 一条脱敏测试消息完成 Archive -> Routing -> ThreadResolver -> Dispatch -> Hermes -> Response Persistence -> Delivery。
- 故障期间积压已处理或有明确、可追踪的处理计划。
- 未通过删记录、改成功状态或绕过 migration 掩盖故障。
- 复盘中产生的正式结论已更新到对应归属文档，讨论过程保留在 Issue 或工单。

## 11. 相关文档

- [项目总纲](../../00_项目总纲.md)
- [部署运维总则](../../04_部署运维.md)
- [CFserver Staging 部署](../deployment/staging-debian.md)
- [V2 Enterprise Runtime 架构总览](../architecture/v2-enterprise-runtime.md)
- [Context Runtime](../context/context-runtime.md)
- [Admin API](../admin/admin-api.md)
- [当前限制](../status/current-limitations.md)
