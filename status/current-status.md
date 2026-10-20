# 当前状态矩阵

> 后续交付补充（2026-10-20）：[独立 Profile B 与管理 Web 隔离结果](../validation/records/2026-10-20-profile-b-management-delivery.md)。四真实 Profile 创建、路由、持续会话、受管模板同步及核心管理 Web 已隔离通过；旧会话即时刷新未接通，真实微信/异机部署未验收。以下矩阵保留 PR13 起点事实，不覆盖新增组件证据，也不表示已部署生产。

> 需求同步：2026-10-20。当前目标以[用户最新确认](../requirements/confirmed-platform-scope.md)及 D044—D053 为准；实现与观察分层。最后生产观察为 **2026-10-19 北京时间 18:36**，本轮未访问生产。

## 状态口径

用户确认功能、固定代码、CI/隔离测试、实际部署、真实模型选工具、现场用户实收和后续Skills分别记录。固定引用见[本轮同步/只读记录](../validation/records/2026-10-20-requirements-sync.md)，历史见[PR11总体证据](../validation/records/2026-10-20-overall-status-evidence.md)与[PR12最终候选证据](../validation/records/2026-10-20-configuration-only-candidate.md)。不以文档HEAD冒充产品、镜像或实时状态。

## 四项目矩阵

| 项目 | 固定源码/分支事实 | CI/隔离证据 | 现场证据与缺口 |
| --- | --- | --- | --- |
| 总文档 | 本轮基于PR11 `462ecc3a16f92e843b8c2276c828fa7b3d159599`，语义吸收PR12 `3aa7bbbbe3ec3a1d756b06bdc5ed2d0265f6a041`；main读取时`dd81075d7516d74a9e307ebdc03ce8ec07e12c49` | 无workflow，本轮仅文档检查 | Draft基于PR11分支；依赖PR9→PR10→PR11，main未同步；旧PR9—12均保留 |
| Gateway | 产品PR19/20/21历史与未合并状态保留；PR21固定`91a235fca08362d8c1440e808df56946be3cc1e8`是本轮只读代码对象 | PR12接续最终PR23 `57bb2c2830f292d5167f0bda449afafbe0a528fd`、PR24 `b3f2740289c6b1fa3208c352bc87b99b6f50cce4`，准确CI见原记录，不重新声称本轮运行 | PDF/PNG/文字/正常ZIP出站有历史现场成功；B派生、Profile生命周期、执行中补充和全档案工具未完成；ZIP入站缺验收 |
| WeChat | 原名PR9 `7bd42b623d405956080a13481bbf4de21aa3477a`、时区PR10 `e9f751819fb8e9bc1772d7f6d67089e5b49f10d7`与后续PR11 `e56c7e4b606f1614e54210a391906f4be7440f5e`分开 | 修复候选构建/隔离与最终组件CI见固定记录 | PDF/PNG/ZIP出站有实收；仍有前缀/中文替换，实际协议部署/原名收件验收未完成；本轮不登录/收发 |
| FileBrowser | PR4/5/6前置栈历史保留；部署PR7最终`47876e3a3d62affe777de8623697f62791fa5fa4`接续PR11的早期候选 | 最终部署资产检查通过，regular后端lint仍111项失败，不能称全仓绿；完整Dockerfile隔离成功不覆盖slim | 单商品归档及受限文件操作、正常ZIP出站可复用；非全库、非所有真实模型操作通过，迁移/restore/完整审计仍分项 |

以上产品PR状态按两份固定记录的原核对时间，本轮不将其当实时远端复核。文档仓库main和PR9—PR12则已本轮通过GitHub核对，详见同步记录。

## 当前平台基础与明确缺口

| 用户已确认需求 | 已有代码/证据 | 当前缺口，不写成已完成 |
| --- | --- | --- |
| 员工Bot权限，再员工群使用权限；授权可未分组 | Identity/source mapping、UserAccessPolicy与准入交集 | 未见独立员工群使用权限字段；群缺绑定可回退unknown_group，与不暗配目标有差距 |
| B：每员工/群独立真实Profile和持续会话 | AgentProfile ref/revision、私聊绑定、GroupType引用、group_sender线程、Hermes session ID续接 | GroupType直接共享引用非模板派生；同群按人隔离与每群目标不同；版本进入线程key需受控处理旧会话 |
| Gateway创建/同步Profile | 仅DB引用/版本和请求字段有基础 | 真实Profile路由、创建、修改同步、旧会话生效分别待验证；目标已确认，不等官方接口或CF已实测 |
| 执行中补充进入当前运行 | 同线程FIFO/RUNNING屏障、会话续接 | FIFO等待不等运行中注入；识别补充/新请求/事后修改及官方API接法未验证 |
| 当前群近文/引用/日期查询 | Message/引用结构、管理档案API、Context工具代码 | Context存储只join成功Dispatch+Response，非全部未@群档案；引用正文注入和受控完整档案工具仍缺 |
| 在线持续采集，人和机器人共用档案 | Message Persist-first、四类时间字段、按identity/conversation/occurred_at筛选、limit/offset | self在Poll跳过，出站另存Response/Delivery，其他来源Bot自发消息不保证；停Poll/范围/窗口/连续性影响完整性 |
| 按任务取文件、明确存/指定资料群例外 | 受控媒体持久化、Attachment、READY Artifact、文件API/工具及限定验收 | 普通文件触发策略需对齐；元数据、临时原件、正式归档分开；离线自动补分类专项撤出 |
| 合法原名保持 | 文件名修复候选与隔离验证 | 实际协议部署及原名用户收件未验收，仍是平台基础必修 |

Profile不是OS沙箱，文件权限仍经唯一File Service；换分组不扩权。`occurred_at`/上游timestamp是发言时间，`received_at`为接入接收时间，`created_at`为建库时间；管理查询主要用occurred_at半开起止区间。北京时间日期/月转明确区间，分页读实际已归档且获准记录，不给模型Admin Token/数据库，也不一次装入全部模型上下文。

## Gateway Web逐项基础

下表来自固定产品源码，页面均未开发；本轮只记需求。状态 API 不等于所需管理写接口或真实采集都存在。

| Web方向 | 已有DB/读取API基础 | 管理写接口/采集缺口 | 页面 |
| --- | --- | --- | --- |
| 员工Bot及群使用权限 | Identity/来源映射、通用UserAccessPolicy | 独立员工群权限与员工授权管理HTTP CRUD缺；不能以SSH/SQL作为正常体验 | 无 |
| 员工分组、微信群分组 | GroupType及群绑定模型 | 目前不是双类模板分组完整模型，管理HTTP CRUD缺 | 无 |
| 模板、B派生Profile/版本/同步/电脑 | AgentProfile ref/revision、路由元数据 | 模板派生、目标电脑、创建同步状态与对应写/读接口缺 | 无 |
| 微信登录及采集/执行/投递状态 | runtime health有auth/Hermes/各worker心跳及任务投递状态 | 需按七项展示关联，不得把登录健康等同采集/执行/投递可用 | 无 |
| 会话/人员/时间档案 | 管理GET messages及过滤/分页 | self/出站/窗口完整性缺口，机器人受控档案工具与权限未完成 | 无 |
| 可接任务电脑、运行/队列/真实资源 | Dispatch/队列/运行状态和有限Hermes心跳 | 无完整主机注册/分组分机管理；未见CPU/内存专用采集 | 无 |
| 员工/群/分组使用频率 | 已存消息/任务可供后续聚合 | 未见专用频率统计采集/聚合API，组维度需补 | 无 |

电脑数量由人数/频率/资源决定，先采用可观察的组到电脑手工配置；B独立Profile落在目标电脑。分机变化的旧会话/在途生效规则待细化，不默认自动阈值、复杂负载均衡或执行中迁移。

## 文件业务验收矩阵

| 独立验收项 | 已有现场证据 | 保留边界 |
| --- | --- | --- |
| 入站取得字节 | PDF正文/原件返回/样本摘要一致，PNG实际读取 | ZIP入站历史超时失败，完整入站未验收，不推断格式不支持/损坏 |
| 内容理解 | PDF正文、PNG准确描述与同会话文字续接 | 非所有格式/商品/多轮通过；运行中补充未验收 |
| 明确保存/分类归档 | 一个商品4主图8详情，原包保留 | 非全库；按任务/指定资料群触发需对齐；路径调整仍暂缓，离线补分类不再专项 |
| 正式库查询回传 | 受限API查询/导出，工具加载后正常商品ZIP重新导出交付 | 客户端不等模型每类自主选用；原样回传不代替按要求保存后独立检索 |
| 原聊天实收 | PDF、PNG图片、正常ZIP；ZIP整包和12成员与收件一致 | PNG按D041满足业务，不再追字节/重编码/file路由；原名仍未通过 |
| 加工结果回存 | 受限文件操作/商品归档/ZIP生成基础 | 通用工作副本/新文件或新版回存/原件关联与完整版本管理未整体Agent验收；细节交Hermes/后续Skills |

## 保留且后置的工作

PR12保留PR23官方安装依赖阶段、Python3.14.7/完整默认[all]环境的原生远程图文和合成ZIP查询导出成功；共享MCP仍缺可信每调用任务授权，未走通Artifact/Response/Delivery。PR24统一入口19阶段及独立Docker、真实HTTPS/业务Token/撤销临时sudo后业务可用、服务重建及重复执行有隔离证据；tmpfs/四项systemd替身不证明统一三组件真实booted systemd、持久宿主盘/整机重启/全新电脑完成。旧单组件clean-device A/B成功不冒充新增统一入口通过。

简部署、统一恢复最终要求保留并后移，不回滚/废弃候选，不继续本轮研发。新恢复保留在线微信登录、离线扫码，与旧forced-QR契约冲突待后续受控修改；本轮脚本未动。结果不明任务不盲重做。Skills在四仓基础完成后由用户找Hermes制作并另建仓库；ERP/平台、群发与高级跨会话检索继续后置。当前群独立Profile/会话与必要上文不是后置高级功能。

原六阶段不变，阶段1仍未整体完成；历史文本、Checkpoint/恢复、受限文件与样本验收不抹掉。PostgreSQL restore、迁移回滚、WebDAV/OnlyOffice、完整服务端跨层审计等没有新证据不标完成。HA/watchdog不捆绑当前交付，阶段1不建独立OCR，本轮不建新Skills Runtime/RAG。
