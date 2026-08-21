# 文档审计与收口报告

> 文档编号：VAL-AUDIT-20260821
> 审计日期：2026-08-21
> 审计基线：`c75a5b0` 及本次收口改动
> 审计范围：本仓库全部已跟踪 Markdown、README、架构、设计、部署、运维、状态与验证材料

## 1. 审计边界

本仓库只包含项目文档，不包含 `CF_agent-gateway`、`CF_agent-wechat`、Hermes 或 `CF_filebrowser-enterprise` 的业务代码。仓库规则同时禁止读取或影响其他仓库。因此本次可以完成：

- 文档之间的当前/历史状态一致性检查。
- 现有验证记录与架构、部署声明的一致性检查。
- 权威入口、相对链接、命令边界、时区和证据字段检查。
- 找出无法追溯到精确代码/镜像的文档声明。

本次不能独立确认外部组件代码是否与文档逐行一致。旧验证记录缺少精确 commit、镜像 digest、Compose hash 和 schema，是明确的可追溯性缺口；不是可以猜测补齐的数据。后续由组件 owner 在新验证记录中提供精确发布输入。

## 2. 盘点结果

| 区域 | 审计结论 | 收口方式 |
| --- | --- | --- |
| 根 README / `00`-`05` | 当前范围和决定基本一致，但缺正式生产文档入口 | README 与 `04` 更新为正式体系导航；`05` 保持决定权威 |
| `architecture/` | 有多个主题页，但总体入口重复、缺 WeChat Runtime 单一设计 | 新增 System Architecture、WeChat Runtime 和目录索引 |
| `design/` | 主要是 2026-08-04 设计/验证快照，部分状态已被后续证据取代 | 保留历史设计，在首屏标明日期和现行状态入口 |
| `docs/architecture/` | 同时含当前摘要和 2026-08-11 快照，直接进入旧页易误读 | `docs/README` 重新分类；旧页增加历史警告；总体摘要改兼容入口 |
| `docs/deployment/operations/status` | Staging 命令和旧“current limitations”与当前五服务/Hermes 状态冲突 | 全部按历史快照隔离，禁止作为生产操作依据 |
| `status/` | 保存关键事实，但“当前摘要”和历史记录重复 | `status/current-status.md` 保持当前能力权威；旧记录加取代提示 |
| 验证体系 | 缺统一 Checklist、执行证据字段和正式目录 | 新建 `validation/`、Checklist、records/history 规则 |
| ADR | 决定集中在 `05_技术决策记录.md`，无正式目录导航 | 新建 `adr/README.md`，不重复复制决定正文 |

## 3. 主要发现

| ID | 严重度 | 发现 | 处理结果 |
| --- | --- | --- | --- |
| DOC-001 | P0 | 缺 System Architecture、Deployment Guide、Recovery Runbook、WeChat Runtime、Production Checklist 和时区规范 | 已新增正式文档 |
| DOC-002 | P0 | 2026-08-11 历史文件自身使用“当前”措辞，直接访问可能误导生产操作 | 保留正文并增加首屏历史警告与现行链接 |
| DOC-003 | P1 | `architecture/ai-system-overview.md`、`docs/architecture/overall-architecture.md` 和 `02_系统设计.md` 形成多重总体入口 | System Architecture 成为企业级入口，旧总体页改为兼容导航，`02` 保留详细设计职责 |
| DOC-004 | P1 | 旧运行手册遗漏当前独立 `wechat-worker`，并可能让人把两套 Compose 当一套操作 | Deployment Guide 强制区分 Gateway/WeChat 两套 Compose 和发布清单 |
| DOC-005 | P1 | 旧 Staging 写 Hermes/WeChat/Workers 未启用，与 2026-08-14 生产证据冲突 | 旧文档按日期隔离；当前能力只看状态矩阵 |
| DOC-006 | P1 | 缺 Host/Container/Database/Application 四层时区基线 | 新增 Timezone Policy 和验证项 |
| DOC-007 | P1 | `localId` 排序/单调性未验证，at-least-once 边界容易被过度承诺 | 正式 Runtime 把 `localId` 定义为 opaque，并把保证限定到入站 Message Store |
| DOC-008 | P1 | 2026-08-14 验证记录缺 commit/digest/config/schema/逐项证据与签核 | 历史记录保持原样；新 records 规范强制补齐 |
| DOC-009 | P1 | 缺 restart/recreate/数据库/Docker/宿主分层恢复操作 | Recovery Runbook 和 Checklist 分层处理，未验证层级保持 BLOCKED |
| DOC-010 | P2 | Task Queue、完整 Context Snapshot 和媒体目标设计可能被总体描述误读为上线 | 正式架构明确使用 Routing/Dispatch 当前链路，目标能力以虚线和状态标注 |

## 4. 缺失、过期和代码一致性结论

### 4.1 已补齐的缺失

- 企业总体架构和组件信任边界。
- 新机器、Docker、配置、Hermes 和微信登录部署流程。
- Docker、微信掉线、QR、Hermes 和消息不回复恢复手册。
- Polling、Checkpoint、`localId`、恢复和 at-least-once 设计。
- restart、消息、Hermes、回复和数据验证清单。
- 四层时区规范。
- ADR、部署、运维、验证正式目录入口。

### 4.2 过期材料

以下材料保留但不是当前生产依据：

- 2026-08-11 `docs/` V2 Enterprise Runtime 架构/部署/运维/限制快照。
- 2026-08-04 `design/` 和 Gateway Staging 验证。
- 早期 `status/agent-wechat-validation.md`。

历史结论不删除、不改写成新事实；首屏警告和索引负责阻止误用。

### 4.3 与代码一致性

本仓库无业务代码，因此没有足够证据宣布“所有命令与当前实现完全一致”。已确认的问题是**发布追溯缺失**：旧记录无法从叙述映射到精确组件 commit、镜像和配置。

正式体系采用以下控制：

1. 组件实现细节和命令由目标版本组件仓库维护。
2. 本仓库只规定跨组件顺序、不变量和占位符结构。
3. 发布前把组件 commit/tag、image digest、Compose 路径/项目、config hash、migration/schema 和 Hermes 控制命令写入发布清单。
4. 未解析占位符阻断部署；不从历史 Staging 或 `main` 猜测。
5. 新 Checklist 记录目标版本的期望、实际与证据，组件 owner 复核后才可宣称一致。

## 5. 仍未关闭的产品/运行缺口

文档收口不会改变以下能力状态：

- Hermes 自启、守护、告警、正式 `uncertain` 管理和 AI 主机恢复。
- 引用正文注入。
- Attachment、媒体私有存储、Hermes 多模态、Artifact 与微信媒体投递。
- 完全新设备 QR。
- Gateway 容器 recreate、PostgreSQL、Docker daemon、CFserver 宿主恢复。
- `group_shared`、File Service 主链、Skills 和企业系统接入。

这些项目必须通过新验证记录后再更新[当前状态矩阵](../status/current-status.md)；不能因为文档现已完整而提升状态。

## 6. 质量检查

| 检查 | 结果 | 说明 |
| --- | --- | --- |
| Markdown 格式 / `git diff --check` | **PASS** | 全仓 53 份 Markdown 的 H1 与围栏检查通过；`git diff --cached --check` PASS |
| 仓库内相对链接与图片路径 | **PASS** | 348 个本地链接/图片路径，0 断链；外部私密 GitHub 链接未联网验证 |
| UTF-8 解码 | **PASS** | 53 个文件严格解码，0 错误 |
| 命令占位符与危险命令 | **PASS** | 占位符必须在发布时解析；WeChat Compose/network create 无硬编码；危险命令仅出现在禁止操作上下文 |
| 敏感信息模式 | **PASS** | Token/私钥模式 0 命中，无真实环境端点；历史文件中的 `0.0.0.0` 仅为禁止公开绑定的泛化示例 |
| Git 状态与变更范围 | **PASS** | 37 个文档文件，仅涉及本仓库 `architecture/`、`deployment/`、`operations/`、`validation/`、`adr/` 及入口/历史警告；无业务代码和其他仓库变更 |

最终检查结果在提交前更新本节。
