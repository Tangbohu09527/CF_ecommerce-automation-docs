# Timezone Policy

> 中文名称：生产时区规范
>
> 文档编号：DEP-002
>
> 文档状态：当前强制基线
>
> 生效日期：2026-08-21
>
> 最近复核：2026-09-03

## 1. 四层基线

| 层级 | 强制时区 | 说明 |
| --- | --- | --- |
| Host | `Asia/Shanghai` | Debian 使用 IANA 名；Windows 使用语义等价的 `China Standard Time` |
| Container | `UTC` | 所有应用容器、Worker 和数据库容器以 UTC 运行 |
| Database | `UTC` | PostgreSQL session/server 时区为 UTC，持久时间使用带时区语义的类型 |
| Application | 内部 UTC，展示转换 | API、事件、日志使用 UTC RFC 3339；界面/报表按用户或业务规则转换，默认 `Asia/Shanghai` |

主机时区便于现场运维；容器和数据库统一 UTC 用于排序、幂等、审计和跨主机关联。应用是唯一负责展示转换的层，不得让数据库或客户端各自猜测。

## 2. Host

### 2.1 Debian

受控配置：

```bash
sudo timedatectl set-timezone Asia/Shanghai
```

验证：

```bash
timedatectl status
date --iso-8601=seconds
date -u --iso-8601=seconds
```

期望：本地时区显示 `Asia/Shanghai`，时钟同步服务健康；UTC 与本地时间表示同一时刻。批准的 `time_source`、`max_clock_skew` 和测量方法由目标发布清单提供，不在本文固定服务名或阈值。

### 2.2 Windows

管理员受控配置：

```powershell
tzutil /s "China Standard Time"
```

验证：

```powershell
tzutil /g
Get-Date -Format o
(Get-Date).ToUniversalTime().ToString("o")
w32tm /query /status
```

`China Standard Time` 是 Windows 的系统 ID；跨平台配置、事件和数据库中仍使用 IANA 语义 `Asia/Shanghai`，不持久化 Windows 专有 ID。`w32tm` 结果应证明批准的时间源、最近同步状态和偏移符合发布清单；若设备由 MDM 管理且不允许使用该命令，必须保存经批准的 MDM 合规证据，证明相同的同步事实。只验证 `tzutil` 不足以证明时钟已经同步。

## 3. Container

Compose 或镜像应按组件支持方式显式设置 UTC。仅看到 `TZ=UTC` 不算通过，必须在运行容器内实测：

```bash
docker exec <CONTAINER_NAME> date -u --iso-8601=seconds
docker exec <CONTAINER_NAME> date +%Z
```

期望时区缩写为 `UTC`，时间带 `+00:00` 或 `Z` 语义。若精简镜像没有 `date`，使用该组件发布文档指定的运行时命令或健康接口验证，不为此临时修改生产镜像。

## 4. Database

PostgreSQL 必须从受控管理连接，以及 Gateway、`wechat-worker`、`dispatch-worker`、`delivery-worker` 实际使用的每个数据库 role/session 连接路径分别执行只读验证：

```sql
SELECT current_user;
SHOW timezone;
SELECT now(), current_timestamp;
```

每份结果记录组件、连接路径、`current_user`、采样 UTC 时间和 `SHOW timezone`；即使多个组件共用一个 role，也要验证各自实际 session。仅证明管理员 session 为 UTC 不足以证明应用 session 为 UTC。期望所有实际 session 的 `timezone` 均为 `UTC`。数据规则：

- 业务时刻使用 `timestamptz` 或等价的 UTC-aware 类型。
- API/event 时间使用 RFC 3339 `Z` 或显式 offset。
- 禁止用无时区 local timestamp 表示绝对时刻。
- 若来源平台提供时间，分别保存来源时间、来源 offset/时区事实和 Gateway 接收 UTC 时间；不得覆盖混合。
- 日期型业务字段如“业务日”需单独定义所属业务时区，不能从 UTC 日期直接截取。

修改数据库时区是受控配置变更，必须通过目标发布的配置/migration 流程完成，不在事故排查时直接执行临时 `ALTER SYSTEM`。

## 5. Application 与展示

- Message、Checkpoint、Admission、Dispatch、Response、Outbox、Delivery Attempt、日志和审计时间内部统一 UTC。
- API 输出优先使用 `2026-08-21T00:00:00Z` 形式；若使用 offset，必须保持语义明确。
- UI、运维报表和面向员工的时间由应用转换为 `Asia/Shanghai`，并显示时区或 offset。
- 排序、过期、重试和幂等窗口基于 UTC 时刻，不基于格式化后的本地字符串。
- 日志关联同时使用脱敏 correlation ID；不要只靠肉眼比较本地时间。

固定验收样例：

```text
输入 UTC:        2026-08-21T00:00:00Z
上海展示:        2026-08-21 08:00:00+08:00
往返 UTC:        2026-08-21T00:00:00Z
```

## 6. 验收与告警

每次新机器部署、数据库恢复和宿主重启后验证：

- Debian/Windows Host 时区与时钟同步。
- Host 使用发布清单批准的 time source，Windows 以 `w32tm` 或批准的 MDM 证据证明同步。
- 每个生产容器实际 UTC 时间。
- PostgreSQL 管理连接和每个应用实际 role/session 的 `current_user` 与 `SHOW timezone` 证据完整，且均为 UTC。
- API/event 保存值为 UTC RFC 3339。
- 应用展示转换通过固定样例和日期边界样例。
- CFserver、AI 主机、数据库和日志系统的时钟漂移低于发布参数 `<MAX_CLOCK_SKEW>`。

`<MAX_CLOCK_SKEW>`、批准 time source 和测量工具/采样方法必须逐次从目标发布清单解析。测量应在同一受控采样批次内记录每个节点的 UTC 采样时间、时间源、offset 和判定结果；不使用人工目测多个终端时间代替漂移证据。

任何时钟回拨、漂移超限、数据库非 UTC 或应用重复转换都阻断生产验证。结果记录在[Production Validation Checklist](../validation/production-validation-checklist.md)。

## 7. 变更规则

- 时区调整必须有 change ticket、影响分析、维护窗口和回滚方法。
- 不把时区变更混入普通服务 restart 或故障排查。
- 改变展示时区不得改写已经持久化的 UTC 时刻。
- 审计记录必须保留原事件时刻与变更操作者，不能通过批量覆盖伪造历史。
