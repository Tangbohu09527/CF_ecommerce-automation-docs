# 2026-10-20 统一部署与官方Hermes纯配置候选证据

## 本轮目标与操作边界

本轮实际开发第一版候选，包含Gateway编排的统一Debian入口、固定官方Hermes原生HTTP MCP验证，以及FileBrowser/WeChat必要的所属仓最小适配。开发模型按用户选择6.1 Sol。最多两条工作线，同一分支单写入者；文档在关键结果后收尾。

没有连接生产服务器、生产API、正式Hermes或真实微信；没有读取生产凭据、企业素材或正式业务目录；没有登录、扫码、UAC、收发开窗、迁移生产、重建镜像或改正式配置。没有创建/恢复运维SSH/sudo授权。本轮CI临时数据、证书、用户和密码只属于一次性隔离环境，不是生产管理窗口。

通过现有已认证gh API按仓库全名读取默认分支、PR base/head/状态、固定源码、实际diff及Actions；没有用搜索摘要或不带ref的默认文件代替候选。四仓独立分支，原工作树/未提交材料保留；不merge、force push、发布正式版本。文档仓只修改Markdown。

## 已核对的四仓起点

| 仓库 | 默认main快照 | 相关固定候选/依赖 |
| --- | --- | --- |
| Gateway | `7963bc0db5a01e38099e021959feabf648db7ede` | PR21 `91a235fca08362d8c1440e808df56946be3cc1e8`，base附件续办 `ef3119aeecd987165255c066218441f6aef8fb0a`；PR22时区 `3410a50250def9d84e34099404662f5e84c8f3d3` |
| FileBrowser | `4750a97cfdf5bd067e04b6b36bf9616f5ada836d` | PR5产品 `3d556786068994826b8105ff4a1d11572b84b1b1`，依赖PR4 `8d12d21e59619c10a794d997edc895a912b58cb5`；PR6时区 `91ad46787bdf005ca18d599c4204a5d01f190cbb` |
| WeChat | `81d21f451adc61df71077ca3035adeddeca4621f` | PR9文件名 `7bd42b623d405956080a13481bbf4de21aa3477a`；PR10时区 `e9f751819fb8e9bc1772d7f6d67089e5b49f10d7` |
| 总文档 | `dd81075d7516d74a9e307ebdc03ce8ec07e12c49` | PR10 `0fe71a67bdc8331df9830ad69247674169b03a9d`，base原PR9 `docs/file-business-workflows-20260927` / `8179533ffade1027d01727cbf5a89a26b491c400` |

原PR保持，不把候选写成main。新分支独立，具体HEAD和CI见后文；本轮文档提交是文档版本，不是产品或生产版本。

## 两个独立兼容组合

| 验证组合 | Gateway源码 | FileBrowser源码 | WeChat与Hermes |
| --- | --- | --- | --- |
| 统一服务器候选 | PR24，当前候选 `71c802da233cfd855d0b7436da543c823a937098`；旧尝试另列 | PR7 `e9fc639d1916eb2d1aa61e869737bee2b38b8601`，基于PR5并集成PR6内容/准备恢复 | 管理PR11 `e56c7e4b606f1614e54210a391906f4be7440f5e`；运行镜像为下列固定upstream digest；Hermes仅配置URL、不调用 |
| 原生Hermes探针 | PR23 `2ada096c8a30ecbde771fb32f7536790709b0cd9`，探针代码 `1f3a1c2978bcc0b4803e6246e309fc77a7865c51` | 固定PR5 `3d556786068994826b8105ff4a1d11572b84b1b1` 的真实HTTP fixture，不是统一安装容器 | 官方 `427d605d4985c2d49a795493ef6c5d66413057d1`；微信未接入，未用生产模型Key |

统一入口选择的协议镜像为 `ghcr.io/thisnick/agent-wechat@sha256:97d5a99f933592c15897a7579b4e2b2cbbb5f4dda856228f8271f91dd7cafb43`；其既有来源观察为协议源码 `96c2e90c497d999bf5189a134a9b3b293731f42e`。管理仓源码含PR9不等于该镜像启用了文件名候选；本轮不切换协议。组件manifest不把不同分支的“各自最新”拼在一起。

## 实际候选变更

- Gateway PR24：`deploy/install-server.sh/.py`、固定组件manifest、受保护输入模板、HTTPS接线、阶段journal和隔离CI。调用三个现有入口，不复制FileBrowser业务。继承PR22的Asia/Shanghai默认，SQL语义UTC；原Gateway执行器只新增可选root-only诊断日志。
- FileBrowser PR7：集成PR6时区默认；准备阶段在任何正式目录写入前保存受保护intent，原随机凭据和文件字节固定；中断后只补缺失且一致的文件，拒绝不明半成品、并发变更和输入漂移，完成后删除intent。新增5项恢复测试。未改产品业务API实现。
- WeChat PR11：现有prepare增加仅root可用的root-managed checkout/configure；不要求新建锁定manager再次输入密码。保留固定来源、路径/身份验证；login venv延后到既有登录入口。未登录、创建runtime、授予sudo或开闸。
- 总文档：更新当前状态/进度、部署导航、需求的后续状态、总纲当前摘要及本记录；六阶段原定义不变，无运行代码或真实企业资料。

原有生产插件/两个EXE未修改。当前机器修通不等于最终“官方Hermes加少量配置”交付。本轮不把新客户端藏进脚本作为替代。

## 官方Hermes原生实测

[固定详细报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/2ada096c8a30ecbde771fb32f7536790709b0cd9/docs/validation/2026-10-20-native-http-mcp.md)、[代码CI39302151607](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39302151607)、[最终HEAD CI39302780142](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39302780142)均已读取，专用检查通过。最终HEAD相对探针代码只补验证结果。root另下载并核对最终run的safe-summary：该次输入摘要为`8b6e2b837740371b0e2e22f2549ba6947b0e92606c114d362f6585474130de49`（ZIP时间戳可变），导出摘要及12成员仍匹配。PR checkout的合成merge `d74305ce72034f74481371a7029521e9cfa569ba`与HEAD的tree均为`ddb7286027febc3718beb5180e7921821ce41f81`；没有把旧代码run当作新HEAD结果。

干净官方HOME只含配置与合成凭据；无项目插件/EXE、PYTHONPATH、源码改动或启动注入。真实官方API两会话创建201、两路请求200；确定性模型替身实际选择原生远程工具，图文进入正式role=tool。FileBrowser真实TLS/HTTP处理器、临时数据库和普通用户执行query/export各两次；fixture source scope为`/`，不能冒称已验更细员工目录矩阵。

代码CI该次输入18,156,370字节，SHA256 `12a212577a83c38bf410e3512036da61955f1af66afe9e0e114f882493db5f29`；导出18,156,942字节，SHA256 `c37cd162e6ec1432a843891685c544a8ada64760cf55e1319a711fc74fa4bb4d`。实际TLS下载字节与导出回执一致，12成员大小/摘要与选定输入一致。重新打包不要求外层ZIP与入站容器相同。

**确切缺口：** 两个会话共享同一MCP transport，8次tools/call的`_meta`均为空。固定官方handler调用`session.call_tool(tool_name, arguments=args)`，未转交API当前任务/会话上下文；identity_header只支持静态或连接时Profile。服务端诊断哨兵实际返回`trusted_task_context_missing`。该哨兵不是已部署Gateway业务端点。

因此尚未进入真实Gateway当前任务的合法返回授权、Artifact/Response和微信替身投递；错身份、过期、跨会话、重复使用的完整端到端矩阵未完成。没有让模型填taskID，没有用共享Admin或移除鉴权。单任务专属受限grant/官方配置进程是最小替代候选，但会新增可信任务生命周期管理；本轮未偷偷实现或当作已满足要求。

最少原生配置形状（仅证明通道，不是可部署完整业务）为官方config中的`mcp_servers.cf.url`、静态受限`headers.Authorization`、`identity_header: {name: X-Hermes-Profile, value_from: profile}`、`platform_toolsets.api_server: [mcp-cf]`，加既有官方模型/API配置。只有profile或静态header不能区分同Profile并发任务。

初次Windows图片写入PermissionError保留；正常权限隔离进程重跑成功，未改ACL。Linux探针曾误识别工具描述/不可信内容包装，修正替身解析真实回执后通过，未删除完整性断言。成功是明确的边界探针成功，不能表述为全业务通过。

## 统一入口验证（结果收尾中）

本地首轮4项输入/journal检查、追加中断镜像回归后共5项检查、Ruff和diff检查通过。PR24专用CI在一次性Debian13容器中调用统一入口，独立真实Docker daemon，不挂host socket；四项systemd只读观察为替身。测试使用独立tmpfs挂载test-disk，不是实际RAID或持久宿主磁盘。未运行booted systemd或整机重启；旧clean-device工作流的单组件A/B不能扩大为新增统一入口B通过。

首次检查 `39303826610` 在 `297729718411886405764e794cb5bc5a8bf46e19` 失败，已下载脱敏result.json及Debian digest。system、依赖、Controller、WeChat checkout/configure/bootstrap、独立凭据、FileBrowser checkout/image实际完成。FileBrowser准备拒绝test harness用默认权限挂出的tmpfs：`unsafe installation parent: /srv/storage`。这不是预定端口冲突，不能算其通过；重试后仍拒绝且没有继续start。

修正提交 `7e3321ad67bee523d52e4df6fdf65617fe7b3550` 仅改测试：tmpfs显式mode0755、预期失败必须实际诊断为port phase，并增加镜像阶段完成记录已写/journal未提交中断的本地回归（5项全部通过）。产品实现未放宽权限，旧失败保留。该次新检查`39305292066`在FileBrowser的lan-inputs阶段正确拒绝测试输入中的loopback CIDR，预期port断言也准确报红；并未继续启动服务。其脱敏错误位置为validator shell line247。

修正`71c802da233cfd855d0b7436da543c823a937098`去除测试loopback CIDR，并在统一输入层提前对齐FileBrowser的规范化unicast CIDR、去重和DNS label契约，本地5项检查通过；还将固定source/image公共回执以严格字段白名单加入脱敏结果，自动push/PR按同分支去重。新准确CI正在运行，最终结果待补。

## 组件准确CI与已知红项

- FileBrowser PR7：deployment-assets `39300420116`通过，涵盖新增准备恢复；本地5项恢复单测通过。regular tests `39300420042`中backend/frontend/format等通过，lint-backend失败111项。产品backend树本轮未改；既存lint不宣称修复。旧FileBridge fixture HTTP410边界保留，未把未触发/未重跑的client矩阵写成新SHA全绿。
- WeChat PR11固定HEAD：component boundary `39300943397`、Debian clean `39300943037`、forced QR `39300943020`、upstream media `39300942997`均通过；无真实微信登录验收。
- Gateway PR21固定产品SHA原8项检查通过，PR22原6项通过。它们不是本轮新入口HEAD CI；本轮准确结果待统一检查后补齐。正常自动push/PR重复及已被新SHA取代的CI允许取消，取消不算通过。
- 总文档无workflow；只做Markdown/链接/敏感内容/空白检查。

## 保留的10月18—19日真实业务（本轮没有重跑）

固定[商品ZIP报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/91a235fca08362d8c1440e808df56946be3cc1e8/docs/validation/2026-10-18-enterprise-product-zip.md)必须读到末尾“Normal WeChat ZIP outbound acceptance — October 19, completed”。

- PDF Message4310/Dispatch49/Delivery44：正文读取、合法引用、Artifact、投递、用户打开和接收摘要一致。额外两工具及持久Skill修改已按原报告记录、用户确认保留；本轮未执行模型或工具补证。
- 文字4311/50/45：同会话接续与用户实收通过。
- 旧PNG4312/51只下载、未read/return，历史未完成保留。新PNG4314/53/attachment6/48实际看图、描述、图片Artifact和用户实收满足使用目标。原PNG文件容器字节差异不再作为阻塞。
- 正常ZIP出站4320/59/50：从已归档正式目录新打包，Artifact `36860c4b-d207-5f6d-a6ce-db337e1a0441`，18,167,063字节、12成员；用户保存件整包与成员一致，整包SHA256 `239fe0ddf1570553ae9fb09639ce3236f09e2fc0af843b7e1f0cc41ed48f3e42`。57仅capture/no Delivery，不是微信失败；4319/58/media7入站超时仍不算成功。
- 文件名有send_file前缀和替换，候选PR9未生产验收。正式归档路径调整暂缓。

## 最后生产观察与本轮最终边界

以下只是原报告10月19日10:36UTC/18:36北京时间观察，本轮没有重新核验：

| 项 | 最后观察值 |
| --- | --- |
| Gateway产品/镜像 | `b0517d2222fcf5a08dc6aed079ffbb8baa133203` / `sha256:0bbbb3c29b8117b000027adda522bdecbfb73c1756d44c08431fd1ee43fc1049`，schema `20261018_01` |
| FileBrowser服务端/镜像 | `d505fc6599608b219f3494979b0a5f197fdf356f` / `sha256:1e6ccdc7ff1854e1a98821a9623136315845a10e5d9a3b5cfbc2d163b4ba5fb0` |
| Hermes/客户端 | 官方 `427d605d4985c2d49a795493ef6c5d66413057d1`；FileBridge `afcd5c875905a849a7636ccc22057b47da419dfe`，现存插件及两个EXE |
| 微信实际制品 | Image ID `sha256:3b14e58c908ec11e9163ef0bbb51522bea15c1568c8e1012a29890b9adadc48e`，registry digest见前文97d5；二进制 `4f07107800d8c6a592f8a0c50bb0cf683e7d454d10b383e87078ed59009715db` |

当时六核心healthy、Poll/Delivery关闭、无在途、Bot已登录仅限该观察时刻；本轮上游待收、队列、登录和服务健康均未访问，不能写当前无在途。原临时授权18:36:04撤销、密码18:36:05删除及负验证有历史记录。本轮没有恢复任何生产管理权限，所以也不为“清理”再连接生产。

## 资料库/Skills与明早最少动作

资料库小批整理与商品查询交付、素材归档、完整性检查Skill准备可与部署收敛并行。已有工具、缺业务规则、缺真实验收分别见[就绪清单](../../status/current-progress.md)。不以单商品成功宣称全企业资料完成，不开发新Skills Runtime/RAG。

无需为本轮补输密码、扫码或UAC。下一轮先审阅候选与原生上下文缺口；统一入口尚缺的booted-host/新机和纯配置完整授权返回链应先隔离补证。真实微信ZIP入站、文件名、正式路径各自保留独立后续，不重派历史任务。群聊多人、完整档案查询、群发、Gateway Web继续排后。
