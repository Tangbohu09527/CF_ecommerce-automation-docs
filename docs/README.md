# CF_agent-gateway V2 Enterprise Runtime 文档

> 适用版本：`v2-enterprise-runtime-20260811`
> 文档状态：当前发布版交接基线
> 状态日期：2026-08-11

## 文档范围

本目录记录 `CF_agent-gateway` V2 Enterprise Runtime 的当前架构、部署、运维、Context Runtime、Admin Archive API 和已知限制，供交接、部署与日常维护使用。

“当前发布版”表示对应能力已经包含在指定版本中，不等于全部服务均已启用或所有外部业务系统已经联调。`CFserver` Staging 当前只确认 PostgreSQL 与 Gateway 已运行；实际部署事实以 [CFserver Staging 部署状态](./deployment/cfserver-staging-status.md) 为准。未在发布事实中确认的接口路径、配置名、端口、重试参数和未来能力，均不得从本文档推断为已实现。

根目录现有 V1 Staging 文档继续保留历史验证和企业总体规划。涉及 `CF_agent-gateway` V2 Runtime 的当前行为与限制时，以本目录的版本化文档为准；固定技术决定仍以根目录的 [技术决策记录](../05_技术决策记录.md) 为准。

## 阅读顺序

1. [V2 Enterprise Runtime 架构总览](./architecture/v2-enterprise-runtime.md)
2. [Staging Debian 部署](./deployment/staging-debian.md)
3. [CFserver Staging 部署状态](./deployment/cfserver-staging-status.md)
4. [Runtime 运维](./operations/runtime-operations.md)
5. [Context Runtime](./context/context-runtime.md)
6. [Admin API](./admin/admin-api.md)
7. [当前限制](./status/current-limitations.md)

## 目录结构

```text
docs/
├── README.md
├── admin/
│   └── admin-api.md
├── architecture/
│   └── v2-enterprise-runtime.md
├── context/
│   └── context-runtime.md
├── deployment/
│   ├── cfserver-staging-status.md
│   └── staging-debian.md
├── operations/
│   └── runtime-operations.md
└── status/
    └── current-limitations.md
```

## 内容归属

| 主题 | 权威文档 |
| --- | --- |
| 模块职责、端到端链路、Gateway 与 Hermes 边界 | [架构总览](./architecture/v2-enterprise-runtime.md) |
| `CFserver` 硬件、目录、Docker 网络、volume 与端口规划 | [Staging Debian 部署](./deployment/staging-debian.md) |
| `CFserver` 当前版本、运行服务、migration 与验证结果 | [CFserver Staging 部署状态](./deployment/cfserver-staging-status.md) |
| 启停、检查、故障处置和恢复顺序 | [Runtime 运维](./operations/runtime-operations.md) |
| Context、Timeline、Snapshot、Memory 的定义 | [Context Runtime](./context/context-runtime.md) |
| Admin Archive API 的只读范围、权限与过滤维度 | [Admin API](./admin/admin-api.md) |
| 已知限制、影响和当前处置边界 | [当前限制](./status/current-limitations.md) |

同一事实只在归属文档中完整维护，其他文档使用链接或简短摘要。发布实现变化时，应同步更新适用版本、当前限制和受影响的运维步骤；固定技术决定发生变化时，先更新根目录的 [技术决策记录](../05_技术决策记录.md)。

## 交接核对

- 部署使用的代码、镜像和配置均对应 `v2-enterprise-runtime-20260811`。
- migration 在 Gateway 和 Workers 启动前成功完成，且没有以跳过 migration 的方式恢复服务。
- 密钥、Token、Cookie、微信登录数据和真实业务文件未进入仓库、普通日志或交接文档。
- `postgres` 不对非必要网络开放，Gateway 与管理查询入口按最小网络范围暴露。
- 当前只确认 `postgres` 与 `gateway` 已运行，数据挂载、migration 和 Gateway readiness 已验证。
- Hermes API 尚未接入；`dispatch-worker`、`delivery-worker` 与 WeChat runtime 均保持未启用状态，未把完整链路写成已完成。
- 运维人员已知当前限制，且没有把 Skill Runtime、Memory Runtime、RAG、WDT 或 S6 集成当作现有能力。
