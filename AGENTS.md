# AI 协作规则

本文件是后续 Codex 和其他代码 AI 在 `CF_ecommerce-automation-docs` 的固定入口。

## 阅读顺序

1. [README.md](./README.md)：仓库边界与摘要。
2. [2026-10-20 最新用户确认与差异表](./requirements/confirmed-platform-scope.md)：当前平台基础优先、B方案独立Profile、分组分机、文件触发、档案/日期查询和Skills后置；优先于相冲突的旧规划。[文件工作流](./requirements/file-workflows.md)保留场景与历史证据，不再把并行Skill试点/离线补分类当当前待办。
3. [当前状态矩阵](./status/current-status.md)：实现、部署、生产验证和剩余边界。该矩阵未更新部分按其日期；文件工作面以新增场景文档与后续带日期证据为补充，不能用旧的部署待办覆盖现场进展。
4. [2026-10-20 总体状态证据](./validation/records/2026-10-20-overall-status-evidence.md)：固定 PR/HEAD/CI、现场源码和镜像、PDF/PNG 与正常 ZIP 出站完成边界。最后现场观察为 10 月 19 日北京时间 18:36，未实时重验；不再把[10 月 11 日历史断点](./validation/records/2026-10-11-overall-status-evidence.md)当待执行步骤。9 月记录保留原日期和限定范围。
5. [00_项目总纲.md](./00_项目总纲.md)与[05_技术决策记录.md](./05_技术决策记录.md)：阶段和固定决定；技术决策优先。
6. 按任务读取需求、系统设计、开发规范、部署运维及根级 `architecture/`、`deployment/`、`operations/`、`validation/`。
7. `docs/`、`design/`、旧 status 记录、XMind 和 PNG 按日期作为兼容或历史材料。

本轮基线、PR11/PR12语义吸收、固定代码只读核对和文档检查见[同步记录](./validation/records/2026-10-20-requirements-sync.md)；PR12 后续统一入口/官方Hermes最终实验保留在[候选证据](./validation/records/2026-10-20-configuration-only-candidate.md)。两份旧记录中的当时下一步不是当前授权。

## 固定技术决定

- 项目名为“电商业务全自动化系统”，相关仓库统一使用 `CF_` 前缀。
- 生产 Agent 使用 Hermes，不引入 OpenClaw。
- 模型计划使用 GPT-5.6 API；不得把计划写成已经正式接入。
- CFserver 是 Debian 部署宿主；Gateway 与 PostgreSQL 是消息、上下文、任务、文件引用、权限、日志和审计关联的状态权威。
- Windows AI 主机负责 Hermes、后续 Skills 和 Windows 侧执行。
- 微信入口使用 external `agent-wechat`；旧 forced fresh QR、`restart: no`、Archive 不自动复用是尚未修改的运行契约。新恢复目标为在线保留登录、离线扫码，与旧契约冲突须后续受控调整验证，不能直接据本文运行脚本。
- Controller 管理 Poll/Delivery Gate；旧契约下 fresh QR 前显式检查并关闭 Gate。恢复入口后置，本轮不执行恢复。
- 第一阶段不建设独立 OCR。
- `CF_filebrowser-enterprise` 是唯一正式 File Service；接管时动态核对部署和限定验收，不把历史“待部署”条目当永远当前状态。
- 生产运行不得持续依赖 GitHub 在线。

固定决定变化须先在[技术决策记录](./05_技术决策记录.md)说明原因、影响和取代关系。当前 D044—D053 调整 D026、D030、D040、D042、D043 等旧目标/时序；D005 持久化、D041 PNG取舍、唯一File Service及获准媒体入站受控中转/出站READY Artifact继续成立。

## 文件工作面的固定约束

- 文件系统保存正式原件/产物；AI 主机加工的是工作副本；明确要求回存时另存新文件或新版并保留原件，未明确要求不自动归档派生稿。
- 普通附件默认不主动处理/回复，不全部下载归档；任务需要才取得原件，明确要求才正式保存，指定纯文档群可配置自动保存例外。消息元数据、任务原件、正式归档分开。
- 分类、别名、手卡、模板与电商流程由后续Hermes/Skills承担；四仓不无限扩建业务逻辑，权限/通道/文件名缺口仍是平台任务。
- AI离线自动补分类专项撤出；已有档案/已取文件/任务状态持久化与防重复保留，离线提示/结束/保留待处理待细化，不默认开机跑全部积压。
- B方案：组是模板，每员工/群独立真实Profile/持续会话；组到电脑由规模和手工配置决定。Gateway创建/同步Profile是目标已确认可行性待验证；字段不等于路由/生命周期已通。
- 四仓基础完成后用户找Hermes做Skills并另建仓库；本轮不创建仓库/Skill/新Runtime或RAG。当前群上文/引用/日期和执行中补充不因高级检索/群发后置而排除。
- PDF 与 PNG 及正常 ZIP 出站已有各自限定验收；PNG 图片识别及微信图片收发满足当前业务要求，原字节差异/重编码/单独 file 路由不再列为阻塞。企业原始资料、ZIP 成员/整包完整性继续保持；ZIP 入站不因出站通过而通过。不为补证重放已成功操作。
- Hermes/Windows 接入不锁死文件服务；最终部署以 D040 和[部署指南](./deployment/deployment-guide.md#最终部署标准与候选边界)为准：服务端单一正式入口、电脑仅官方 Hermes 加少量配置，无额外项目 EXE/手工复制插件/源码修改/专用常驻进程，不依赖工作树或旧回执。当前插件桥接与开发验证中候选分别记录。

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
| 本轮最新需求正文/差异 | `requirements/confirmed-platform-scope.md`；决定编号仍只在 `05_技术决策记录.md` |
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
