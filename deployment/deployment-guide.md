# Deployment Guide

> 中文名称：生产部署指南
>
> 文档编号：DEP-001
>
> 文档状态：当前部署基线
>
> 最近复核：2026-08-21
>
> 运行证据截止：2026-08-14

## 1. 适用范围

本文用于在新 CFserver 和 Windows AI 主机上部署阶段 1 系统，并用于已有环境的可追踪升级。它规定跨仓库顺序、配置和验收门禁；实际 Compose 文件、服务名、migration、微信登录脚本和 Hermes 进程控制命令由目标发布版本的组件文档提供。

当前存在两个重要限制：

- 完全新设备的 SSH 二维码登录尚未实机验证。
- AI 主机重启后的 Hermes 可靠自启、守护和告警尚未完成生产验收。

因此，新机器部署只有在相关测试记录为 PASS 后才能标记为生产完成；文档存在不等于该能力已经验证。

## 2. 发布输入

每次部署先创建脱敏发布清单。以下字段不得留空或保留尖括号占位符：

| 字段 | 要求 |
| --- | --- |
| `release_id` / change ticket | 唯一、可追踪，可关联审批与验证记录 |
| 文档基线 | 本仓库精确 commit |
| Gateway 版本 | `CF_agent-gateway` 精确 commit/tag 和镜像 digest |
| WeChat 版本 | `CF_agent-wechat` 精确 commit/tag 和镜像 digest |
| Gateway Compose | 文件路径、project 名和 service 名；配置 hash 只针对经批准的非敏感模板或脱敏后的 canonical config |
| WeChat Compose | 固定版本中的文件路径、project 名和 service 名；配置 hash 只针对经批准的非敏感模板或脱敏后的 canonical config |
| 数据库 | PostgreSQL 镜像 digest、schema/migration 版本和备份/恢复命令 |
| Hermes | 版本、安装包校验和、配置档案引用、健康地址和批准的启停/守护命令 |
| 网络 | `cf-internal` 定义、允许的 CFserver 到 AI 主机端点；不记录凭证 |
| 时间同步 | 批准的 `time_source`、`max_clock_skew`、测量工具/命令、采样方法和证据位置 |
| Secret 引用 | 受控 Secret 路径或标识，不记录 Secret 值或指纹 |
| 验证 | Checklist 版本、操作者、复核人、维护窗口和回滚负责人 |

配置 hash 不得基于含 Secret 的完整渲染输出，也不得把 Secret hash 或指纹写入发布记录。canonical config 的脱敏和规范化方法必须版本化，确保复核人可以用同一方法重算。GitHub `main` 链接只用于导航，不能作为可复现发布输入。部署包、镜像、配置模板和校验和应提前进入受控离线存储；生产运行不得依赖 GitHub 持续在线。

## 3. 环境要求

### 3.1 CFserver

- Debian 主机，具体受支持版本以目标发布清单为准。历史 Staging 中的 Debian 和 Docker 版本只是当日观测，不是最低支持版本。
- Docker Engine 与 Docker Compose Plugin，版本已由对应组件目标版本验证。
- PostgreSQL 和 Gateway 持久数据使用明确的 named volume 或受控 bind mount；路径与备份范围写入发布清单。
- 主机时区为 `Asia/Shanghai`，容器为 UTC；实际 time source、最大允许漂移和测量方法与发布清单一致。
- 足够的磁盘容量、日志轮转、备份空间和恢复窗口。
- `cf-internal` 只连接获准组件；PostgreSQL 不暴露到公网。
- 到 Windows AI 主机 Hermes 端点的受控网络路径。

基础只读检查：

```bash
timedatectl status
docker version
docker compose version
docker info
docker network inspect cf-internal
```

命令成功只证明运行环境存在，不证明组件版本兼容。兼容性必须由发布清单确认。

### 3.2 Windows AI 主机

- 受支持的 Windows 版本与补丁基线由 Hermes 发布输入确认。
- 主机时区语义为 `Asia/Shanghai`；Windows 系统 ID 使用 `China Standard Time`。
- Hermes Gateway 当前验证基线为 0.20.0；新版本必须重新验证，不得从文档日期推断兼容。
- Hermes 配置档案由 Hermes 创建和管理；Gateway 只保存 `external_profile_ref`。
- 必须定义批准的服务控制、守护、健康探测、告警和重启后自动恢复方式。
- 模型/API 凭证通过受控 Secret 注入，不进入仓库、普通 YAML、命令输出或截图。

### 3.3 操作工作站

- 能访问受控发布包、目标主机和验证记录存储。
- 终端历史、日志采集和屏幕共享不会记录 Secret、二维码或微信登录材料。
- 操作者具备最小必要权限，生产变更具有第二人复核。

## 4. 配置基线

| 配置项 | 生产要求 |
| --- | --- |
| `ENABLE_VNC` | `0`；生产不使用 VNC、noVNC、x11vnc、websockify 或宿主桌面 X11 |
| `bootstrap_mode` | 新环境首次建立高水位时才可使用 `latest`；恢复已有环境禁止重新 bootstrap |
| 内部网络 | Gateway 与 `agent-wechat` 使用 `cf-internal`；仍必须 Token 鉴权 |
| Secret | root-only 文件、未提交的 `.env` 或受控 Secret 系统；不得写入普通 Compose/YAML |
| Gateway 数据库 | UTC、持久化、纳入备份；schema 与发布版本一致 |
| Hermes Profile | 清单中的 `external_profile_ref` 必须在 Hermes 存在且与预期档案匹配 |
| 媒体存储 | 未完成 Attachment/Artifact 生产接入前，不配置为已启用能力 |
| 正式文件 | 只经 `CF_filebrowser-enterprise` File Service、权限检查和审计 |
| 日志 | UTC 时间、脱敏关联 ID、轮转和保留期；不记录正文、凭证或真实账号 |

详细时区规则见[Timezone Policy](./timezone-policy.md)。

## 5. 新机器部署

### 5.1 建立部署记录

1. 创建 `release_id` 和维护窗口。
2. 填完发布清单并由第二人复核；任何未解析占位符都阻断部署。
3. 记录目标主机、当前数据是否为空、备份要求和回滚目标。
4. 核对目标发布的组件权威文档：
   - [`CF_agent-wechat` 生产部署](https://github.com/Tangbohu09527/CF_agent-wechat/blob/main/docs/deployment/cfserver-production.md)
   - [`CF_agent-wechat` 登录管理](https://github.com/Tangbohu09527/CF_agent-wechat/blob/main/docs/login-management.md)
   - [`CF_agent-gateway` 生产部署](https://github.com/Tangbohu09527/CF_agent-gateway/blob/main/docs/deployment/cfserver-production.md)
   - [`CF_agent-gateway` 微信 Runtime](https://github.com/Tangbohu09527/CF_agent-gateway/blob/main/docs/runtime/wechat-runtime.md)

使用这些链接定位文档后，部署记录仍必须写精确 commit/tag；不能让链接随 `main` 漂移。

### 5.2 准备主机与持久化

1. 按[Timezone Policy](./timezone-policy.md)配置并验证主机时间。
2. 安装发布清单批准的 Docker Engine 与 Compose Plugin。
3. 创建发布定义要求的目录、owner、权限、volume 和备份目标。
4. 确认 `cf-internal` 的定义。仅当发布清单明确它是 external network 且当前不存在时，才执行批准的创建命令：

```bash
<APPROVED_NETWORK_CREATE_COMMAND>
docker network inspect cf-internal
```

创建后将 inspect 结果与发布清单逐项比较：driver、subnet、IPv6、labels、`attachable` 和 `internal` 必须完全符合批准定义；不一致时停止部署，不通过删除并猜测重建来修复。

5. 将 Gateway 与 WeChat 的目标版本部署包放入独立目录，不从另一个仓库复制 Compose 文件。
6. 通过受控方式放置 Secret；权限和 owner 检查结果进入部署记录，但不记录内容。

### 5.3 渲染检查

Gateway 与 WeChat 是两套 Compose，必须分别检查：

```bash
docker compose --env-file <GATEWAY_ENV_FILE> -f <GATEWAY_COMPOSE_FILE> -p <GATEWAY_PROJECT> config --quiet
docker compose --env-file <WECHAT_ENV_FILE> -f <WECHAT_COMPOSE_FILE> -p <WECHAT_PROJECT> config --quiet
```

目标 Compose 版本必须先确认支持 `config --quiet`。不要把完整 `docker compose config` 输出写入普通日志，因为渲染结果可能包含 Secret。命令中的所有占位符必须先从发布清单解析。

检查项：

- 镜像解析到发布清单中的 digest，而不是可漂移 tag。
- volume/bind mount 指向预期持久化位置。
- PostgreSQL 没有公网端口。
- `agent-wechat` 为 `ENABLE_VNC=0`。
- Gateway 三个 Worker 均存在：`wechat-worker`、`dispatch-worker`、`delivery-worker`。
- 两套 Compose 都连接 `cf-internal`，但使用各自 project 与 Secret。

### 5.4 部署顺序

以下是职责顺序，具体 service 名与命令由发布清单展开：

1. 拉取或导入锁定 digest 的镜像，核对校验和。
2. 启动 PostgreSQL，等待发布定义的 readiness。
3. 仅当本次版本需要时，在备份和维护窗口内执行批准的 migration；普通 restart 不盲跑 migration。
4. 启动 Gateway API，验证数据库和 readiness。
5. 在 Windows AI 主机安装并启动锁定版本的 Hermes，创建或核对配置档案。
6. 从 CFserver 和 Windows AI 主机分别验证 `<HERMES_HEALTH_URL>`。
7. 启动 `agent-wechat`，使用目标版本的登录管理脚本确认现有 session 或执行受控登录。
8. 启动 `wechat-worker`、`dispatch-worker`、`delivery-worker`。
9. 执行健康、消息和回复验证；未通过前不扩大员工或群范围。

Compose 操作形式如下，实际参数必须来自发布清单：

```bash
docker compose --env-file <GATEWAY_ENV_FILE> -f <GATEWAY_COMPOSE_FILE> -p <GATEWAY_PROJECT> pull
docker compose --env-file <GATEWAY_ENV_FILE> -f <GATEWAY_COMPOSE_FILE> -p <GATEWAY_PROJECT> up -d <APPROVED_GATEWAY_SERVICES>
docker compose --env-file <WECHAT_ENV_FILE> -f <WECHAT_COMPOSE_FILE> -p <WECHAT_PROJECT> pull
docker compose --env-file <WECHAT_ENV_FILE> -f <WECHAT_COMPOSE_FILE> -p <WECHAT_PROJECT> up -d <APPROVED_WECHAT_SERVICES>
```

`up -d` 可能创建或 recreate 容器。当前 Gateway 容器 recreate 尚未完成生产恢复验收，所以已有生产环境执行前必须按变更风险单独审批。`docker compose restart` 不应用新镜像或配置，不能用来完成版本发布。

### 5.5 微信登录

- 只使用目标 `CF_agent-wechat` 版本的权威登录管理脚本。
- 保持 `ENABLE_VNC=0`，通过受控 SSH 终端显示二维码；二维码、Cookie 和 session 文件按 Secret 管理。
- 不截图、不复制到 Issue/聊天、不写入普通日志。
- 手机确认后分别验证微信登录状态、受 Token 保护的 API 和脱敏测试文本。
- 完全新设备二维码流程当前为“未验证”，必须在[生产验证清单](../validation/production-validation-checklist.md)形成 PASS 记录后才能解除门禁。

### 5.6 Hermes

本仓库没有已经验证的通用 Hermes 绿色安装命令，因此不得虚构 service 名、端口或启动脚本。发布清单必须提供：

- 安装包来源和校验和。
- 版本检查命令。
- 配置档案创建/核对方式。
- 受控启停、守护和健康检查命令。
- CFserver 可访问的健康端点。
- Windows 重启后自动恢复验证。

仅存在 Windows 登录启动项不构成自启证据。若重启恢复测试未通过，该部署只能处于受控验证状态，不能标记为完整生产就绪。

## 6. 部署后验证

先完成只读检查：

```bash
docker compose --env-file <GATEWAY_ENV_FILE> -f <GATEWAY_COMPOSE_FILE> -p <GATEWAY_PROJECT> ps
docker compose --env-file <WECHAT_ENV_FILE> -f <WECHAT_COMPOSE_FILE> -p <WECHAT_PROJECT> ps
docker network inspect cf-internal
```

然后执行[Production Validation Checklist](../validation/production-validation-checklist.md)，至少确认：

- Host/Container/Database/Application 时区符合基线。
- PostgreSQL、Gateway API 与三个 Worker 达到发布定义的健康状态。
- `agent-wechat` 登录、内网 Token 鉴权和文本接口正常。
- Hermes 的版本、健康、Profile 引用和 CFserver 网络路径正确。
- 获准私聊、群聊真实 `@`、群聊未 `@`、未授权身份和 Bot 防回环符合预期。
- Response Persistence、Delivery Outbox、Delivery Attempt 和微信实际接收可按同一脱敏关联 ID 串联。

图片、文件、`group_shared`、完全新设备登录或宿主恢复未通过时，必须记录 BLOCKED/NOT_RUN，不能用文本健康检查替代。

## 7. 升级与回滚

### 7.1 升级前

- 备份 PostgreSQL、必要的 Gateway 私有状态、微信登录数据和加密配置。
- 验证备份可读，并记录恢复命令与目标；“完成备份”不等于“已验证恢复”。
- 比较 schema、Compose、volume、Secret 引用、Profile 和网络差异。
- 明确 migration 是否可回滚，以及应用与 schema 的兼容窗口。
- 记录当前镜像 digest、配置校验和和健康基线。

### 7.2 失败处理

- 在首个未通过门禁处停止，不扩大影响范围。
- 应用/镜像回滚与数据库回滚分开执行；不得假设旧应用兼容新 schema。
- 不执行 `docker compose down -v`，不删除 volume、微信 session 或 Checkpoint。
- 不直接修改数据库伪造成功状态。
- Dispatch 已进入 `uncertain` 时转[Recovery Runbook](../operations/recovery-runbook.md)，不通过重跑整条消息来恢复。

### 7.3 完成条件

只有发布清单、备份、部署日志、完整 Checklist、残余风险和审批签核都已归档，部署才可标记完成。运行事实进入新的 `validation/records/` 记录；不得修改历史记录来覆盖失败。

## 8. 禁止操作

- 使用未锁定的 `latest` 镜像或漂移的 `main` 作为生产发布输入。
- 将真实 Secret 写入命令行示例、普通 YAML、Git、日志或截图。
- 把 2026-08-11 Staging 文档中的版本、服务或命令直接复制到生产。
- 在普通 restart 中顺带执行 migration。
- 用 `restart` 声称已经应用新配置，或用 `up -d` 隐瞒 recreate 风险。
- 删除 volume/session、手改数据库或盲重试 `uncertain` Dispatch。
- 在生产启用 VNC/noVNC 或让组件绕过 File Service 访问正式文件。

## 9. 相关文档

- [System Architecture](../architecture/system-architecture.md)
- [Timezone Policy](./timezone-policy.md)
- [Recovery Runbook](../operations/recovery-runbook.md)
- [部署运维总入口](../04_部署运维.md)
- [当前状态矩阵](../status/current-status.md)
