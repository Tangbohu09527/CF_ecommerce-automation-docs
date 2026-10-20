# 2026-10-20 最新需求确认与总文档同步记录

> 范围：只修改 `CF_ecommerce-automation-docs` 的 Markdown，必要只读固定代码、文档检查、普通提交/推送及本轮 Draft PR。需求来源为本轮用户确认，不以历史聊天或旧待办增补授权。开发模型使用用户选择的 6.1 Sol。

## 1. 工作树与远端基线

原主检出 `D:\Tangbohu09527\CF_ecommerce-automation-docs` 读取时为 clean/main `c75a5b097da9687140289fba0f718ad2281e8710`。先读适用 AGENTS、Git status、remote、branch 和 worktree list；没有在原检出切分支或覆盖材料，也没有在父目录初始化 Git。旧工作树全部保留。

用户指定的 `D:\Tangbohu09527\.cf-worktrees\docs-profile-b` 原不存在，已从 PR11 精确 HEAD 新建本轮独立分支 `codex/docs-confirm-profile-b-20261020`。这是文档工作树，不是第五业务仓库或部署依赖。

2026-10-20 通过 `gh api`/`gh pr list` 核对，随后仅在总文档仓库 fetch 相关引用：

| 对象 | 实际 base 分支与 SHA | 实际 head SHA | 状态 / 相对 base 提交数 |
| --- | --- | --- | --- |
| main | 默认分支 | `dd81075d7516d74a9e307ebdc03ce8ec07e12c49` | 尚未包含本轮及PR9—12文档栈 |
| PR9 | main `dd81075d7516d74a9e307ebdc03ce8ec07e12c49` | `8179533ffade1027d01727cbf5a89a26b491c400` | OPEN/Draft；3 |
| PR10 | `docs/file-business-workflows-20260927` / `8179533ffade1027d01727cbf5a89a26b491c400` | `0fe71a67bdc8331df9830ad69247674169b03a9d` | OPEN/Draft；1 |
| PR11 | `codex/docs-overall-status-20261011` / `0fe71a67bdc8331df9830ad69247674169b03a9d` | `462ecc3a16f92e843b8c2276c828fa7b3d159599` | OPEN/Draft；1，无给定HEAD后的新增 |
| PR12 | `codex/docs-overall-status-20261011` / `0fe71a67bdc8331df9830ad69247674169b03a9d` | `3aa7bbbbe3ec3a1d756b06bdc5ed2d0265f6a041` | OPEN/Draft；4，无给定HEAD后的新增 |

PR11分支为 `codex/docs-overall-status-20261020`；PR12分支为 `codex/docs-config-only-delivery`。远端 branch list 中没有适用的新整合分支。`git merge-base` 证实两者共同基线为PR10 `0fe71a6…`；本轮没有从过时main重写，也不把PR12整份覆盖PR11。

PR12 的4个提交按先后为 `12adf12`、`e0d01f4`、`314a42b`、`3aa7bbb`，最后一项是历史未完成检查澄清。它们均已纳入差异审阅，未停在早期候选结果。main/PR核对使用既有连接和正常工具审批；普通沙箱请求曾被本地受限代理拒绝，随后正常提权只读成功，没有改代理或重建认证。

## 2. PR11/PR12语义吸收及合并依赖

| 来源 | 本轮保留或吸收内容 | 边界 |
| --- | --- | --- |
| PR11固定HEAD | 总体范围、原六阶段、D040—D043原行、总体现场记录、PNG取舍、文件验收/恢复导航 | 旧决定原日期保留，新增D044—D053说明调整，不假装新功能实现 |
| PR12最终HEAD | 新增 `2026-10-20-configuration-only-candidate.md` 完整正文保留，仅新增顶部来源说明；最终候选/依赖实验/CI/回执和成功失败边界；当前摘要接续到最终SHA | 未对其当时“并行Skills/明早部署/群聊最后”安排续授权；当前目标以本轮确认正文为准 |
| 本轮用户确认 | 平台基础、B方案、Profile管理待验证、补充消息、文件触发/原名、档案/日期、Web与分组分机、后置/撤出项及差异表 | 集中写在[需求正文](../../requirements/confirmed-platform-scope.md)，其他入口简短同步并链接 |

PR12原记录的Git blob为 `c64cb353b6a24b15d6becc90284300e690b00cf7`。本轮仅在顶部新增历史来源说明；去除该说明后与PR12原文件逐字节一致，保证没有覆盖其后续最终结果。PR11既有带日期验收记录未修改，D001—D043原行逐行保留。

后续Draft base为PR11分支 `codex/docs-overall-status-20261020`；合并依赖为 PR9 → PR10 → PR11 → 本轮。PR12与PR11是PR10之上的兄弟分支，其新结果已语义吸收；旧PR12仍保留，未来如独立合并需处理重复/冲突，不自动关闭或改base。**本轮Draft和推送不是main已同步。**

## 3. 必要只读代码核对

没有扫描所有旧聊天、提交或旧工作树材料。Gateway旧主检出只读状态为clean/detached `29b8c5e5d41fc482c3c445069cd664fddcafdc7d`；WeChat旧主检出为clean/main `92393bc2ae1d89dae9449fc131413979aa2fa2f2`，落后本地origin/main 45提交。两仓只列已有worktree，不修改；没有把旧检出当当前产品。

以下使用本地完整固定对象 [Gateway PR21 `91a235fca08362d8c1440e808df56946be3cc1e8`](https://github.com/Tangbohu09527/CF_agent-gateway/tree/91a235fca08362d8c1440e808df56946be3cc1e8) 的 `git show` / `git ls-tree` 核对，路径均相对其仓库。结果不等于远端此刻main、生产镜像或现场验收。

| 核对项 | 固定路径 / 行 | 可证明与不能证明 |
| --- | --- | --- |
| Profile元数据与创建 | `src/cf_agent_gateway/agent_profile/models.py:44–88`；`agent_profile/store.py:40–87`（同一src根） | DB保存key/revision/ref；create_agent_profile只写Gateway DB，不创建真实Hermes Profile |
| 请求字段及session | `src/cf_agent_gateway/routing/resolver.py:54–88`；`src/cf_agent_gateway/hermes/client.py:98–161` | 请求携带profile_reference/revision和session header；不证明真实Profile路由/创建/同步/旧会话生效 |
| B模板与群语义 | `src/cf_agent_gateway/agent_profile/models.py:94–130`；`src/cf_agent_gateway/workspace/thread_keys.py:97–118` | GroupType直接引用Profile，group_sender含sender、group_shared不含；key含profile/revision，不是B派生/持续群会话已实现 |
| 未分组 | `src/cf_agent_gateway/agent_profile/store.py:142–147、313–325` | 私聊无绑定失败；群可回退unknown_group，必须与本轮未分组不暗配的目标并列 |
| 员工权限 | `src/cf_agent_gateway/identity/models.py:26–68`；`src/cf_agent_gateway/access/policy_models.py:55–80`及`evaluator.py:43–63` | 身份、通用用户权限/范围/Skills交集及群@检查；未见独立员工群使用权限已完成证据 |
| FIFO和会话续接 | `src/cf_agent_gateway/task/model/store.py:548–575`；`src/cf_agent_gateway/hermes/service.py:254–263` | 线程先前阻塞或其他RUNNING时不领取，保存Hermes session；不等于执行中补充注入 |
| 留档与self缺口 | `src/cf_agent_gateway/ingestion/service.py:123–131`；`src/cf_agent_gateway/adapters/wechat/polling_service.py:1204–1273` | sink内先存Message后准入；Poll的self不进sink，未@可存与Bot全部自发消息完整存档是不同结论 |
| 日期/分页 | `src/cf_agent_gateway/admin/routes.py:92–140`；`src/cf_agent_gateway/admin/store.py:119–121、235–237` | GET支持身份/会话/起止，limit≤100/offset；occurred_at按>=start、<end，需授权分页及北京时间换算 |
| 时间字段 | `src/cf_agent_gateway/message/models.py:136、151–157` | timestamp/occurred_at为上游发言时间/规范化；received_at接收；created_at建库，不能混用 |
| 当前上下文工具 | `src/cf_agent_gateway/context/tool.py:97–130`；`src/cf_agent_gateway/context/storage.py:293–306` | 有近期/日期工具，但存储只join成功Dispatch+Response，非全部群未@消息或完整Message档案 |
| Web管理/采集基础 | `src/cf_agent_gateway/admin/routes.py`；`src/cf_agent_gateway/runtime/health.py:108–140`及该固定tree | 档案GET、dispatch恢复、心跳/auth/Hermes/任务基础；未见员工/分组/Profile管理HTTP CRUD或前端、完整主机注册/CPU内存/频率专用采集 |

[WeChat原名候选固定报告](https://github.com/Tangbohu09527/CF_agent-wechat/blob/7bd42b623d405956080a13481bbf4de21aa3477a/docs/validation/2026-10-18-return-filename-candidate.md)继续区分合法basename修复候选、隔离验证与实际协议部署/用户收件验收；后两者未完成。

PR23最终`57bb2c2…`/PR24最终`b3f2740…`不在此次本地代码对象中，本轮不为它们下载代码或重跑实验，结果复用PR12固定完整记录，不宣称本轮独立远端复验。FileBrowser未做新增代码遍历；其已有事实复用固定文档。

## 4. 文档检查

2026-10-20 对本轮36份Markdown改动执行本地只读文档校验，结果如下；不是产品测试、CI或现场验收。

| 检查 | 实际结果 |
| --- | --- |
| UTF-8、替换字符、单H1、围栏、冲突标记、行尾空白 | 36份改动全部通过；无非Markdown改动 |
| 相对链接/锚点 | 改动文档350条相对链接、17个锚点通过；全仓61份Markdown合计497条相对链接、22个锚点均通过，包含指向本轮更名/调整章节的反向引用 |
| 决策编号、引用与表格 | D001—D053连续唯一，全仓决策引用均存在；53行表格连续，未被空行截断 |
| 历史保留 | PR11原D001—D043决策行逐行相同；既有带日期验收文件未改；PR12除顶部来源说明外原正文逐字节相同 |
| 敏感内容 | 新增/改动内容7类凭据/私钥/连接串/内网地址模式无命中，并人工核对为脱敏证据和固定SHA，无真实凭据/业务文件 |
| `git diff --check 462ecc3a16f92e843b8c2276c828fa7b3d159599` | exit 0，无空白错误 |
| 独立文档复核 | 需求与决定覆盖、当前入口旧语义、PR12最终结果交叉复核完成；发现并修正决策表空行、派生稿默认回存措辞、历史来源说明和后置工程标记 |
| 文档CI | 本地固定tree没有`.github/workflows`，远端该目录API返回404；无文档workflow，不宣称CI全绿 |

检查只解析Markdown和读取Git对象；未运行产品测试、安装、模型调用、可行性实验或现场验收。组件CI仅复用PR11/PR12固定记录的实际结论，未触发或重跑。提交/推送后的SHA和Draft身份由Git/GitHub回读，不用本记录自身文字代替远端结果。

## 5. 生产与历史边界

最后实际观察仍是 **2026-10-19 北京时间18:36**：当时收发关闭、无在途、原Bot登录，随后临时授权撤销、临时密码删除。原PDF/PNG、单商品归档、受限操作、正常ZIP出站与历史失败均保留，不改写为本轮验收。

本轮未访问CFserver或正式Hermes、未读取生产密码、未申请SSH/sudo、未弹UAC、未操作微信/开关收发/调用业务模型/安装脚本，未建真实Profile或改Hermes配置。未修改其他三仓的源码、文档、配置、分支或PR；未触碰CF_S6、`_workspace`、`D:\新谷力`。普通文档Git写入/推送仅限本轮独立分支，旧材料和工作树全部保留。
