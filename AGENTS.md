# AI 协作规则

本文件是后续 Codex 和其他代码 AI 在 `CF_ecommerce-automation-docs` 的固定入口。

## 阅读顺序

1. [README.md](./README.md)：仓库边界与摘要。
2. [文件业务场景与实施优先级](./requirements/file-workflows.md)：2026-09-27 用户确认的文件工作主线。查找/微信回传、共享群真实附件分类归档、AI 主机工作副本加工后 API 回存优先；不要继续把直接修改/删除当主要工作，也不要把终端逐单审批作为正常收件体验。
3. [当前状态矩阵](./status/current-status.md)：实现、部署、生产验证和剩余边界。该矩阵未更新部分按其日期；文件工作面以新增场景文档与后续带日期证据为补充，不能用旧的部署待办覆盖现场进展。
4. [2026-09-17 Gateway 长任务回传](./validation/records/2026-09-17-gateway-long-task-acceptance.md)：Gateway 限定证据；[2026-09-03 Production Closeout](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md)保留为历史总体基线。未更新组件仍按原日期，不把旧镜像当新发布。
5. [00_项目总纲.md](./00_项目总纲.md)与[05_技术决策记录.md](./05_技术决策记录.md)：阶段和固定决定；技术决策优先。
6. 按任务读取需求、系统设计、开发规范、部署运维及根级 `architecture/`、`deployment/`、`operations/`、`validation/`。
7. `docs/`、`design/`、旧 status 记录、XMind 和 PNG 按日期作为兼容或历史材料。

## 固定技术决定

- 项目名为“电商业务全自动化系统”，相关仓库统一使用 `CF_` 前缀。
- 生产 Agent 使用 Hermes，不引入 OpenClaw。
- 模型计划使用 GPT-5.6 API；不得把计划写成已经正式接入。
- CFserver 是 Debian 部署宿主；Gateway 与 PostgreSQL 是消息、上下文、任务、文件引用、权限、日志和审计关联的状态权威。
- Windows AI 主机负责 Hermes、后续 Skills 和 Windows 侧执行。
- 微信入口使用 external `agent-wechat`；forced fresh QR、`restart: no`，Archive 不自动复用。
- Controller 管理 Poll/Delivery Gate；Host reboot 后 fresh QR 前必须显式检查并关闭 Gate。
- 第一阶段不建设独立 OCR。
- `CF_filebrowser-enterprise` 是唯一正式 File Service；接管时动态核对部署和限定验收，不把历史“待部署”条目当永远当前状态。
- 生产运行不得持续依赖 GitHub 在线。

固定决定变化须先在[技术决策记录](./05_技术决策记录.md)说明原因、影响和取代关系。文件业务需求细化继续遵循唯一 File Service、Gateway 入站媒体持久化和出站 READY Artifact 边界，不通过绕过消息准入或直接挂载正式目录实现。

## 文件工作面的固定约束

- 文件系统保存正式原件/产物；AI 主机加工的是工作副本，默认新文件或新版回存、保留原件。
- 商品图、质检、手卡、授权等资料按业务规则分类；未知/冲突进入待确认，不能猜造商品归属、报告字段或授权有效性。
- 预授权资料群的常规新增归档目标是自动执行；覆盖、删除、跨授权范围发送另行确认。群/身份/目录规则未确定前不得擅自全局放行。
- AI 主机可离线，附件先在 CFserver 收妥再排分类任务是待实现/待验收目标，不把元数据当已取得原文件。
- 先验证 PDF/图片真实字节往返，再做群归档和工作副本；不反复新建同一文本测试文件，也不为补证重放 apply。
- Hermes/Windows 的接入适配不锁死文件服务；新机一条命令部署仍是目标，不能把现场补丁序列称作已交付。

## 事实优先级

1. 组件代码、测试、合并 commit 和 GitHub 状态。
2. 带日期的真实生产验收记录。
3. 组件正式文档。
4. 本仓库权威文档。
5. 历史记录。
6. 设计、规划和推断。

跨仓库事实使用获准的 GitHub 只读查询核对 SHA/PR/CI，不访问其他仓库的本地工作区。未合并组件 PR 不写成 main；固定文档提交可引用但须标明其合并状态。合并源代码、观察到的 Image ID、registry digest、完整构建来源和实机验收分别记录，不能互相推导替代。

## 文档归属

| 内容 | 位置 |
| --- | --- |
| 范围、阶段、设备职责 | `00_项目总纲.md` |
| 用户场景、系统行为、验收标准 | `01_功能需求.md`；文件业务细化见 `requirements/file-workflows.md` |
| 职责、数据流、状态和模型 | `02_系统设计.md` |
| 开发与协作规则 | `03_开发规范.md` |
| 部署、健康、备份和排障 | `04_部署运维.md` |
| 固定技术决定 | `05_技术决策记录.md` |
| 当前能力状态 | `status/current-status.md` |
| 当前进度 | `status/current-progress.md` |
| 带日期证据 | `validation/records/` |

组件细节在组件仓库完整记录，总文档只保留摘要、固定引用和跨系统边界。

## 操作边界

- 本仓库只修改 Markdown；不修改业务代码、配置、Compose、Workflow、许可证、二进制附件或其他仓库。
- 不连接、部署、启停或修改 CFserver/AI 主机，不执行生产 Docker、数据库或 SSH 操作。
- 不创建或合并生产 PR/Tag，不改写公开分支历史；文档变更使用独立分支，合并需另行批准。
- 不提交密钥、密码、Cookie、Token、QR、微信登录数据、真实账号/会话、内网 IP、数据库 URL 或业务文件。
- 当前状态不保存动态 Message/Checkpoint/Queue/container/heartbeat/Archive 计数；这些仅进入带日期记录。
- 不把回复自报耗时当作独立工具证据，不把限定长任务通过扩大为所有会话/重启/高可用通过。
- 修改后检查 UTF-8、单 H1、围栏、Mermaid、相对链接、图片、敏感信息、空白和 Git 状态；CI 按实际提交/run 记录。
