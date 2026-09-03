# AI 协作规则

本文件是后续 Codex 和其他代码 AI 在 `CF_ecommerce-automation-docs` 的固定入口。

## 阅读顺序

1. [README.md](./README.md)：仓库边界与当前摘要。
2. [当前状态矩阵](./status/current-status.md)：实现、部署、生产验证和剩余边界。
3. [2026-09-03 Production Closeout](./validation/records/2026-09-03-enterprise-runtime-production-closeout.md)：当前生产证据和组件 Git 基线。
4. [00_项目总纲.md](./00_项目总纲.md)与[05_技术决策记录.md](./05_技术决策记录.md)：阶段和固定决定；技术决策优先。
5. 按任务读取需求、系统设计、开发规范、部署运维及根级 `architecture/`、`deployment/`、`operations/`、`validation/`。
6. `docs/`、`design/`、旧 status 记录、XMind 和 PNG 按其日期作为兼容或历史材料。

## 固定技术决定

- 项目名为“电商业务全自动化系统”，相关仓库统一使用 `CF_` 前缀。
- 生产 Agent 使用 Hermes，不引入 OpenClaw。
- 模型计划使用 GPT-5.6 API；不得把计划写成已经正式接入。
- CFserver/PostgreSQL 是消息、上下文、任务、文件引用、权限、日志和审计关联的权威控制中心。
- Windows AI 主机负责 Hermes、后续 Skills 和 Windows 侧执行。
- 微信入口使用 external `agent-wechat`；生产行为为 forced fresh QR、`restart: "no"`，Archive 不自动复用。
- Gateway Runtime Controller 管理 Poll/Delivery Gate；Host reboot 后 fresh QR 前必须显式检查并关闭 Gate。
- 第一阶段不建设独立 OCR。
- `CF_filebrowser-enterprise` 是唯一正式 File Service；V1 Beta 已实现和自动化验证，但尚未生产部署与集成。
- 生产运行不得持续依赖 GitHub 在线。

固定决定如需变化，先在[技术决策记录](./05_技术决策记录.md)新增决定，说明原因、影响和取代关系。

## 事实优先级

1. 组件当前代码、测试、合并 commit 和 GitHub 状态。
2. 带日期的真实生产验收记录。
3. 组件仓库当前正式文档。
4. 本仓库当前权威文档。
5. 历史验证记录。
6. 设计、规划、XMind 和推断。

跨仓库事实必须通过明确获准的 GitHub 只读查询核对精确 SHA、PR 和 CI；不得访问其他仓库本地工作区。未合并组件 PR 不能写成 `main` 权威，组件计划不能写成系统已完成。

## 文档归属

| 变更内容 | 主要位置 |
| --- | --- |
| 范围、阶段、设备职责 | `00_项目总纲.md` |
| 用户场景、系统行为、验收标准 | `01_功能需求.md` |
| 组件职责、数据流、状态、数据模型 | `02_系统设计.md` |
| 开发和仓库协作规则 | `03_开发规范.md` |
| 部署、健康、备份和排障 | `04_部署运维.md` |
| 已拍板技术选择 | `05_技术决策记录.md` |
| 当前能力状态 | `status/current-status.md` |
| 当前进度 | `status/current-progress.md` |
| 带日期生产证据 | `validation/records/` |

同一信息只在归属文档完整记录，其他位置使用链接或简短摘要。

## 操作边界

- 只允许修改本仓库 Markdown；不得修改业务代码、配置、Compose、Workflow、许可证、二进制附件或其他仓库。
- 不连接、登录、部署、停止或修改 CFserver/AI 主机，不执行 Docker、数据库或 SSH 操作。
- 不创建或合并生产 PR、Tag，不改写公开分支历史。
- 不提交密钥、密码、Cookie、Token、QR、微信登录数据、真实账号/会话、内网 IP、数据库 URL 或业务文件。
- 当前状态不保存动态 Message/Checkpoint/Queue/container/heartbeat/Archive 计数；这些只进入带日期记录。
- 修改后检查 UTF-8、单 H1、围栏、Mermaid、相对链接、图片、敏感信息、行尾空格和 Git 状态。
