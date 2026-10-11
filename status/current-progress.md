# 当前进度与下一步

> 总体状态同步：2026-10-11。业务需求沿用 2026-09-27；仓库证据通过 GitHub 只读核对，现场事实来自用户回传，本次未连接生产。
>
> 完整 SHA、PR、CI 与现场截止点见[10 月 11 日证据摘要](../validation/records/2026-10-11-overall-status-evidence.md)，能力分层见[当前状态矩阵](./current-status.md)，文件业务继续以[原需求](../requirements/file-workflows.md)为准。其他文档任务未完成时不等待合并，也不引用其尚未完成成果。

## 当前工作流

阶段 1 消息 Runtime 已有历史生产交付，文件基础业务仍在联调。9 月 25—27 日三个组件已有同机部署和限定健康、受限读取及受控文本新建证据；这些不是 10 月 11 日重启后的状态。本次保留原六阶段与 PR #9，不重新建立一套需求。

2026-09-27 用户明确：主线是微信查询/分发资料，资质共享群附件识别分类归档，以及 AI 主机下载工作副本、加工、回存。直接改原件或删除需求较少，不再按通用 CRUD 把这两项排在真实附件业务之前。

## 2026-10-11 当前接续点

- 已完成：旧 f9 两端回退、d9 Controller `complete-rollback`、旧核心恢复；随后 d9 新候选镜像和两端新 plan 均通过。
- 未执行：现存新 deployment 的 `apply/start/verify`；原聊天实际附件实收未验收。deployment 与镜像标识只在[带日期摘要](../validation/records/2026-10-11-overall-status-evidence.md#3-用户-10-月-11-日现场事实与截止点)集中记录。
- 重启前：旧 Hermes/Gateway API/Dispatch 运行，Poll/Delivery 停止，Gate 关闭、数据库正常。
- 最新事件：用户明确已执行 CFserver reboot；重启后状态未核验，Windows 是否关机未确认。不得沿用历史 PID，也不得把重启前健康写成重启后恢复。

当前三个候选均未合并：Gateway d9 修复分支无关联 PR，原 Draft PR #14 仍为 f9；WeChat 9b 媒体分支无关联 PR；FileBrowser bb11 位于未合并 PR #2 功能栈。Gateway/WeChat 对应 CI 通过，FileBrowser 专项通过但常规 lint-backend 红项仍在；CI 和隔离测试不代表候选已经现场运行。

## 文件工作面已完成及证据等级

- 2026-09-25 至 09-27 的用户输出显示 FileBrowser 与微信/Gateway 同机运行，AI 主机 HTTPS 和受限凭据接入完成；这是限定现场部署记录，不是正式发布或所有客户端联调通过。
- 2026-09-26 微信真实文件读取：完整内容、SHA-256 及客户端审计关联通过。
- 新建客户端 `28a4481fc2f73363f60bb0f6821bfc50817e99c3` 已由用户安装、启用；Hermes API 已显示原读取工具及可选新建工具。该源码属于未合并功能线，不写成 main。
- 2026-09-27 用户完成计划、终端批准、微信触发创建及创建后 read/checksum；53 字节正文与三处摘要一致。用户随后回传本地 plan/approval/attempt/outcome 与已有 CLI 审计的只读关联核验通过。
- 2026-09-27 记录中的 Hermes 更新、切回本地模式和重启后的配置/工具检查及随后功能有现场结果；不保证任意未来版本或所有重启场景兼容。
- 原始 Hermes trace、FileBrowser Persistent Audit 及完整跨层关联仍未独立核验。客户端记录不是服务端审计，既有操作不能为补证再执行。

- 2026-10-06 FileBrowser bb11 固定记录补充：既有 PDF/PNG 经真实 HTTPS 下载，官方 Hermes 本机原生读取/理解有独立比对；不证明微信入站附件或完整 Agent 会话。
- 2026-10-07 Gateway d9 所含固定记录补充：微信文字请求经 File API 下载副本并读取、回原聊天文字；不等于原附件发回微信，模型传输图片字节未观察到。

## 下一步优先级

0. **先核验重启后与现存计划。** 后续现场任务核对 CFserver 核心、数据库、Controller/Gate、Windows/Hermes 可达性及现存两端 plan；fresh QR 前显式检查并关闭组合 Poll/Delivery Gate，Archive 不复用。确认目标、候选和基线仍匹配后，才接续尚未执行的切换与 verify。不从头回退、传包、构建，不重放已完成步骤。当前文档任务不执行这些动作。

1. **真实附件往返。** 用非敏感 PDF 与商品图样本验证实际字节获取、受控入库、查找与微信原附件回传。原件发送与图片消息分开验收，不能把缩略图或文件名当原件。
2. **指定资质共享群自动归档。** 明确群/发送人/目录准入，再实现分类、商品匹配、待确认、去重和来源记录。常规已授权新增不逐份要求终端批准；不全局取消普通群未 @ 的安全边界。
3. **AI 主机工作副本。** 通过 FileBrowser API 下载、加工、检查结果、另存回库并关联原件；只分发原件时不强制绕行 AI 主机。
4. **后续收口。** 当前先交付微信真实文件/图片收发，不并行扩安装器或大重构；之后按业务价值补正式替换/删除、跨层审计、断线/离线恢复、备份恢复、CI 红项与统一安装。终端/微信来回审批不是最终日常使用体验。

AI 主机离线期间由 CFserver 先收妥原件、分类任务稍后续接，是目标而非当前能力保证。现有附件元数据不证明内容已保存。具体共享群、目录字典和自动归档规则尚未配置，本次文档更新不改变生产权限。

验收分成六项独立结果：微信入站取得字节、内容理解、指定目录/分类归档、文件中心查询回传、原聊天实际附件实收、加工结果回存。文本新建、读取成功、HTTP 200、CI 绿不能相互代替，详见[分项清单](../validation/production-validation-checklist.md#b11-媒体文件和业务扩展)。

轻量 Gateway Web 仅列待规划：局域网 IP 访问、Gateway 权限管理、查询已经接收的部分聊天/任务记录；不承诺微信完整历史，不复制 FileBrowser 文件权限体系。本轮不开发、不部署。旺店通/S6 等按原有证据保留后续集成范围。

## 2026-09-17 Gateway 历史限定成果

- Gateway PR #11 修复已合并，固定合并基线 `9a1caa237a9053678c80f68fdb15d351d5bfecf8` 已核对。
- 现场新应用 Image ID、Dispatch read/execution 600 秒和停止宽限期有证据；完整 source/image provenance 不因此自动成立。
- uncertain 任务经带审计 mark_dead 保留原错误；既有排队只读任务完成，Dispatch 与组合 Poll/Delivery Gate 经受控恢复。
- 分钟级任务服务端派发耗时 113.799 秒，持久响应、单次投递和微信实收匹配；原始工具日志及其他可靠性专项未由此完成。

详见[长任务回传记录](../validation/records/2026-09-17-gateway-long-task-acceptance.md)。Gateway 文档 PR #12 合并提交为 `7963bc0db5a01e38099e021959feabf648db7ede`；本仓库文档同步不表示再次部署。

## 其他历史里程碑与持续边界

Gateway V2/P1、组件文档收口、forced-QR R2 promotion、私聊/真正 @ 群聊、未 @ 结束、Checkpoint 回退、自回复跳过、核心重启与 fresh QR、Context/Admin recovery 等分别保持原证据日期，不写成本轮重复测试。

- 历史会话连续性告警仍单独跟踪；用户暂缓处置，除非重新影响当前链路，不插回文件业务主线。不删队列或重置 Checkpoint 清告警。
- automatic boot stop gate 未证明，CFserver reboot/fresh QR 仍走正式组合 Controller；agent-wechat 的 restart:no 和 Archive 不复用规则不变。
- AI host-only reboot、未触碰微信的 Gateway-only 变更不当然需要重扫，但仍须检查状态；没有新增无人值守恢复保证。
- 真实 PDF/图片文件业务、群自动归档、工作副本及全类型媒体未因文本读写测试而交付。
- 备份恢复/回滚、WebDAV/OnlyOffice 真实联调、发布来源、CI 门禁及 PR 合并仍需逐项收口；部分绿灯不代表全仓库全绿。
- 企业 Skills Runtime、旺店通/S6、RAG 等未由本次接入。第一阶段不建设独立 OCR。

技术决定仍以[技术决策记录](../05_技术决策记录.md)为准，文件业务需求细化不绕过 D012/D018 的唯一 File Service、D027 的入站媒体受控持久化或 D028 的 READY Artifact 投递。
