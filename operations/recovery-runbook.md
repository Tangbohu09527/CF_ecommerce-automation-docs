# Recovery Runbook

> 中文名称：生产故障恢复手册
>
> 文档编号：OPS-001
>
> 文档状态：当前运行手册
>
> 最近复核：2026-08-21
>
> 运行证据截止：2026-08-14

## 1. 使用范围

本文处理以下故障：

- Docker 服务异常或需要受控 restart。
- 微信掉线。
- 微信 QR 重新登录。
- Hermes 异常或不可达。
- 微信消息没有回复。

操作者必须持有目标环境的发布清单、`release_id`、两套 Compose 参数、Hermes 管理命令和受控日志权限。本文中的占位符未解析时停止操作，不猜测文件、服务名、端口或数据库状态。

## 2. 核心原则

1. **先保存事实，再改变现场。** 记录故障时间、范围、版本、健康状态和脱敏关联 ID。
2. **从入口到投递逐段定位。** 不因“没有回复”直接重启全部服务。
3. **状态不明不重做。** Dispatch `uncertain` 禁止盲重试。
4. **执行与投递分离。** Response 已持久化时只恢复 Delivery，不重新调用 Hermes。
5. **不同恢复层级分开。** service restart、容器 recreate、数据库重启、Docker daemon 重启和宿主重启不是同一测试。
6. **不破坏持久化。** 不删除 volume、Checkpoint、微信 session、Outbox 或审计记录。
7. **不手改成功状态。** 数据库直接修改不能成为常规恢复方式。
8. **不泄露。** 日志、截图和事故记录不包含消息正文、真实微信账号、二维码、Cookie、Token、连接串或本地绝对路径。

## 3. 标准排查链

消息无回复时，先用脱敏来源消息键（`sourceAccount + chatId + localId` 的受控引用或 hash）和窄 UTC 时间窗定位 `agent-wechat` 来源事实。首次 Message Store 入库后取得 Gateway correlation ID，再用该 ID 自前向后串联后续节点；来源消息尚未入库时不能假设已经存在 correlation ID。

```text
微信
  ↓
agent-wechat
  ↓
wechat-worker
  ↓
Gateway（Message Store → Identity/Policy/Admission → Routing）
  ↓
dispatch-worker / Dispatch
  ↓
Hermes
  ↓
Response Persistence → Delivery Outbox
  ↓
delivery-worker / Delivery Attempt
  ↓
原微信会话
```

简化责任链是：

```text
微信 → wechat-worker → Gateway → dispatch → Hermes → delivery
```

必须定位第一个不满足不变量的节点。后段健康不能证明前段消息曾到达，前段成功也不能证明后段已经送达。

## 4. 事故初始动作

### 4.1 建立记录

记录以下脱敏信息：

- `incident_id`、`release_id`、环境、操作者和复核人。
- 故障首次发现的 UTC 与 `Asia/Shanghai` 时间。
- 影响范围：单消息、单会话、微信入口、Gateway、Hermes 或全系统。
- 各仓库 commit/tag、镜像 digest、Compose 配置校验和、数据库 schema 和 Hermes 版本/Profile。
- 尚未入库时记录脱敏来源消息键和窄 UTC 时间窗；首次入库后补充 Gateway correlation ID。
- Message、Dispatch、Response、Outbox 的脱敏关联 ID；不记录正文和原生账号。
- 最近变更、已执行动作和当前是否仍有新消息进入。

### 4.2 只读检查

使用发布清单展开以下命令：

```bash
docker compose --env-file <GATEWAY_ENV_FILE> -f <GATEWAY_COMPOSE_FILE> -p <GATEWAY_PROJECT> ps
docker compose --env-file <WECHAT_ENV_FILE> -f <WECHAT_COMPOSE_FILE> -p <WECHAT_PROJECT> ps
docker network inspect cf-internal
docker compose --env-file <GATEWAY_ENV_FILE> -f <GATEWAY_COMPOSE_FILE> -p <GATEWAY_PROJECT> logs --since <UTC_START> --no-color <SERVICE>
docker compose --env-file <WECHAT_ENV_FILE> -f <WECHAT_COMPOSE_FILE> -p <WECHAT_PROJECT> logs --since <UTC_START> --no-color <SERVICE>
```

先按脱敏来源消息键和窄 UTC 时间窗查 `agent-wechat`；若来源事实不存在，继续停留在入口层排查。确认首次 Message Store 入库后，取得 correlation ID 并只用该 ID 检查 Admission、Routing、Dispatch、Response 和 Delivery。日志只在受控终端查看；导出前脱敏。不要运行会渲染 Secret 的完整 Compose 配置输出。

### 4.3 严重不变量

发现以下任一情况立即停止自动处理并升级：

- 非 self 消息没有 Message Store 记录，但对应 Checkpoint 已越过。
- 未授权或未真实 `@` 的消息调用了 Hermes。
- 同一来源消息产生多个独立 Dispatch 或多个业务副作用。
- `uncertain` 被自动重新执行。
- Response 未持久化却已投递，或 Response 已存在但恢复动作准备重跑 Hermes。
- 非 `READY` Artifact 进入投递。
- 数据库、volume 或微信 session 被删除/替换。
- 日志或事故材料出现 Secret、二维码或真实业务正文。

## 5. Docker restart

### 5.1 适用判断

先区分操作级别：

| 操作 | 语义 | 当前验证边界 |
| --- | --- | --- |
| `docker compose restart <service>` | 重启现有容器，不应用新镜像或配置 | 仅 Gateway 应用服务 restart 的有限恢复已有证据 |
| `docker compose up -d` | 可能创建或 recreate 容器并应用配置/镜像 | Gateway 容器 recreate 尚未验收 |
| PostgreSQL restart | 数据库进程和连接恢复 | 尚未验收 |
| Docker daemon restart | 影响同宿主多个容器和网络 | 尚未验收 |
| CFserver reboot | 影响容器、网络、挂载、登录和时钟 | 尚未验收 |

不要用低层级验证结果代替高层级操作审批。

### 5.2 先止损与留证

1. 判断是否仍有新消息进入；若需要冻结入口，使用目标版本批准的 maintenance/worker stop 命令，不删除 Checkpoint。
2. 保存 `ps`、容器镜像 digest、restart count、健康状态、近端日志和积压指标。
3. 核对 volume 与 bind mount 仍指向发布清单位置。
4. 查询是否存在 `running`、`uncertain` Dispatch 或未完成 Delivery，记录关联 ID。
5. 数据库操作前确认备份与恢复目标。

### 5.3 允许动作

仅在没有配置或镜像变更、持久化位置正确且影响明确时，执行单服务 restart：

```bash
docker compose --env-file <ENV_FILE> -f <COMPOSE_FILE> -p <PROJECT> restart <APPROVED_SERVICE>
```

恢复顺序原则：

1. PostgreSQL 最先恢复并通过 readiness。
2. Gateway API 恢复并核对 schema。
3. Hermes 健康和网络路径确认。
4. `agent-wechat` 登录和 API 确认。
5. `wechat-worker`、`dispatch-worker`、`delivery-worker` 按发布定义恢复。

如需 `up -d`、数据库/Docker daemon 重启或宿主 reboot，转变更流程并使用对应的未验证恢复测试，不把它伪装成普通 restart。

### 5.4 禁止动作

- `docker compose down -v` 或任何 volume 删除。
- 删除/重建微信 session 作为第一反应。
- 切换到未锁定镜像或重新拉取 `latest`。
- 在 restart 时顺带运行 migration。
- 为清除积压直接修改数据库状态。
- 一次重启两套 Compose 和 Hermes 后再判断故障点。

### 5.5 恢复验证

- 所有目标服务达到发布定义的健康状态。
- Checkpoint 从原值连续推进，没有重新 bootstrap。
- 重读消息被唯一键吸收，没有重复 Dispatch。
- 原 Workspace、AI Thread、Hermes Thread/Profile 绑定保持。
- 原 Outbox 只投递一次，未重跑已完成 Hermes 执行。
- 运行[生产验证清单](../validation/production-validation-checklist.md)中的对应 restart 层级并归档证据。

## 6. 微信掉线

### 6.1 先分类

按顺序区分：

1. `agent-wechat` 容器未运行或不健康。
2. `agent-wechat` API 不可达或 Token 鉴权失败。
3. 容器/API 健康，但微信 session 失效。
4. 微信登录正常，但 `wechat-worker` Polling 失败或 Checkpoint 停滞。

### 6.2 处理步骤

1. 保存 WeChat Compose `ps`、近端日志、登录状态和 `wechat-worker` 指标。
2. 核对 `cf-internal` 与 Token 引用；不把网络可达等同于登录成功。
3. 容器异常且无配置变化时，可按 Docker 单服务 restart 流程处理。
4. session 失效时转[QR 重新登录](#7-qr-重新登录)，不要先删除 session。
5. 登录健康但 Polling 失败时，检查 API 分页、超时、`localId`/Checkpoint 作用域和上游保留窗口。
6. 恢复期间保留已有数据库和 Checkpoint，禁止启用 `bootstrap_mode=latest` 跳过积压。

### 6.3 恢复验证

- 微信登录状态与受 Token 保护的消息 API 均通过。
- `wechat-worker` 从原 Checkpoint 继续，重复消息只命中已有记录。
- 若上游保留窗口已越过，显式记录同步缺口并升级，不能宣称零丢失。
- 用脱敏私聊和群聊场景验证收取；未通过前不恢复完整流量。

## 7. QR 重新登录

### 7.1 前置条件

- 已确认是微信 session 问题，而不是容器、网络、Token 或 Worker 问题。
- 目标版本 `CF_agent-wechat` 登录管理文档和脚本已写入发布清单。
- 具有受控 SSH 终端和获准手机确认人。
- 已评估上游消息保留窗口和离线期间积压。
- 发布清单已经明确批准的企业 Bot、Source Account Mapping、session 持久化位置、Checkpoint namespace 和目标会话范围。

### 7.2 操作

1. 使用批准的 maintenance/worker stop 方式暂停新的微信采集，不删除 Checkpoint。
2. 保持 `ENABLE_VNC=0`；生产不启用 VNC/noVNC/x11vnc/websockify。
3. 运行目标版本权威登录脚本，在受控 SSH 终端显示二维码。
4. 二维码按 Secret 处理：不截图、不录屏、不粘贴到聊天或 Issue。
5. 手机确认后等待脚本明确返回登录成功，再核对 API。
6. 恢复采集前，将实际登录的企业 Bot、Source Account Mapping、session 持久化位置、Checkpoint namespace 和目标会话逐项与发布清单比较。任何账号、映射或 namespace 不一致都立即阻断，不修改映射迁就错误登录。
7. 全部一致后恢复 `wechat-worker`，从已有 Checkpoint 继续；禁止重新执行首次 bootstrap。

完全新设备 QR 流程当前尚未实机验证。若脚本、设备确认或 session 持久化行为与既有环境不同，停止并升级，不通过临时开启远程桌面绕过门禁。

### 7.3 恢复验证

- 容器 restart 后 session 持久化符合组件版本预期。
- 登录状态、Token 鉴权、文本读取和文本发送通过。
- 离线窗口中的消息按上游保留能力恢复；任何缺口被记录。
- Bot 回复不回环。
- 二维码、Cookie 和 session 未进入日志或证据附件。

## 8. Hermes 异常

### 8.1 现象

- CFserver 到 Hermes 健康端点超时或拒绝连接。
- Hermes 进程未运行、版本/Profile 不匹配或响应错误率异常。
- Dispatch 停滞、明确失败或进入 `uncertain`。
- Response 未形成，微信没有回复。

### 8.2 只读检查

1. 记录受影响 Dispatch 的状态和脱敏关联 ID。
2. 在 Windows AI 主机按发布清单检查 Hermes 进程、版本、健康和 Profile。
3. 从 CFserver 检查 `<HERMES_HEALTH_URL>`，区分进程故障与网络故障。
4. 查询是否已有 Hermes 响应证据、Gateway Response、Outbox 或 Delivery Attempt。
5. 核对最近配置、凭证、模型限流和主机重启事件。

### 8.3 允许动作

- Hermes 明确未运行且没有结果不明执行时，使用发布清单中的 `<HERMES_SERVICE_CONTROL>` 启动或 restart。
- 恢复后先验证版本、健康、Profile 和 CFserver 连通，再允许新的 Dispatch。
- 明确失败按产品定义处理；`uncertain` 进入独立受控流程。

### 8.4 `uncertain` 处理

1. 停止该 Dispatch 的自动重试。
2. 核对 Hermes 线程/响应、Gateway Response、Outbox、Delivery 与外部业务副作用证据。
3. 使用正式管理命令/API和幂等 Guard 做出“确认已执行、确认未执行、继续等待或人工处置”决定。
4. 记录操作者、证据、决定和结果。

当前正式通用 `uncertain` 管理能力尚未完成。历史上一次带备份、证据核对和 Guard 的人工恢复只证明个案可审慎处理，不是允许手改数据库的 SOP。没有正式工具时，保留证据并升级，不盲重试。

### 8.5 恢复验证

- Hermes 版本、Profile、健康探测、守护与告警符合发布清单。
- 新的脱敏文本测试只执行一次并形成 Response。
- 原有 `uncertain` 没有被自动重做。
- AI 主机 reboot 后自动恢复只有在独立测试 PASS 时才能标记完成。

## 9. 消息不回复

### 9.1 先判断是否为预期无回复

以下场景当前设计为不产生机器人回复：

- 身份未映射或 User/Gateway Policy 拒绝。
- 群聊没有结构化事实证明真实 `@` 当前机器人。
- `is_self=true` 的 Bot 自发消息。

这些场景应有对应的保存/跳过证据，不是可用性故障。不要为了“让它回复”绕过 Admission。

### 9.2 逐段定位

| 节点 | 检查 | 失败处理 |
| --- | --- | --- |
| 微信 / `agent-wechat` | 登录、来源消息可读、API、Token | 转微信掉线或 QR 流程 |
| `wechat-worker` | Polling、标准化、Message Store、Checkpoint | Checkpoint 越过未持久化消息时立即升级 |
| Gateway Admission | Identity Mapping、两级策略、真实 mention | 拒绝为预期；配置错误走权限变更，不临时放行 |
| Routing | Workspace、AI Thread、Thread Policy、`external_profile_ref` | 修复配置前停止派发；不使用隐式默认绕过 |
| Dispatch | 是否创建、明确失败或 `uncertain` | `uncertain` 转证据核对，不重跑消息 |
| Hermes | 进程、网络、Profile、响应证据 | 转 Hermes 异常流程 |
| Response | 是否先持久化、关联 Dispatch 是否唯一 | 未持久化不得直接发送微信 |
| Delivery | Outbox、Attempt、`delivery-worker`、目标会话 | Response 已存在时只恢复 Delivery，不重跑 Hermes |
| 微信实际接收 | 原会话是否唯一送达、是否重复 | 核对 Attempt/上游回执，停止重复发送 |

### 9.3 媒体消息

当前媒体 AI 和回传链路尚未生产完成。图片能够被发现和提取不表示 Hermes 能看图或微信能收到 AI 文件。排查时分别核对来源二进制、Attachment、私有存储、Hermes 媒体协议、Artifact `READY`、媒体 Outbox、`send_media` 和实际接收；未实现节点记录 BLOCKED，不用文本路径伪造成功。

### 9.4 恢复验证

- 使用同一脱敏 correlation ID 串联 Message、Admission、Routing、Dispatch、Response、Outbox 和 Attempt。
- 获准消息在原会话只收到一次回复。
- 拒绝、未 `@` 和 self 场景没有 Hermes 调用。
- Delivery 重试没有创建第二个 Hermes Dispatch。
- 事件和日志时间符合[Timezone Policy](../deployment/timezone-policy.md)。

## 10. 升级条件

以下情况由值班负责人升级到组件 owner、安全或数据恢复负责人：

- 需要 container recreate、数据库/Docker daemon/宿主重启，而该层级尚未验收。
- 需要 migration、schema 回滚或备份恢复。
- `uncertain` 且无法证明外部执行是否发生。
- Checkpoint/Message Store 不变量破坏或疑似丢消息。
- 重复业务副作用、发错会话、越权调用或 Secret 泄漏。
- 完全新设备 QR、上游保留窗口缺口或未形成正式管理工具的故障。
- 问题涉及 `CF_filebrowser-enterprise`；本仓库任务不得修改该仓库。

## 11. 关闭事故

事故关闭前完成：

- 根因、影响范围、开始/恢复 UTC 与本地时间。
- 全部操作与审批、组件版本、关联 ID 和脱敏证据。
- 消息丢失/重复、权限、业务副作用和数据一致性判断。
- 对应 Checklist 结果与残余风险。
- 后续修复 owner、截止时间和需要更新的 ADR/设计/部署文档。

运行记录新增到 `validation/records/`，历史记录不可改写；需要改变固定技术决定时先更新[技术决策记录](../05_技术决策记录.md)。
