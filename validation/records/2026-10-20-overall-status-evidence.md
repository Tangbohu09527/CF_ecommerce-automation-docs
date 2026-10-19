# 2026-10-20 总体状态跨仓证据摘要

> 文档同步：2026-10-20；GitHub 证据冻结时间：2026-10-20 02:28（北京时间，Asia/Shanghai）。本轮仅通过已认证 GitHub 读取四仓 PR、固定文件/必要差异与准确 SHA 的 CI，不读取三个组件本地工作区，不连接生产、不运行模型或微信。
>
> 最后生产状态是报告观察的 **2026-10-19 北京时间 18:36**，不是本轮实时核验。报告私有原始日志、凭据、收件原文件没有在本轮重新读取；此处引用已发布的脱敏记录，不把私有路径或摘要索引当作已亲自检查原件。

## 1. 文档基线、分支和合并依赖

| 对象 | 本轮核对结果 |
| --- | --- |
| 文档 main | `dd81075d7516d74a9e307ebdc03ce8ec07e12c49` |
| [PR9](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/9) | OPEN/Draft；base main，上述 SHA；head `docs/file-business-workflows-20260927` / `8179533ffade1027d01727cbf5a89a26b491c400` |
| [PR10](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/10) | OPEN/Draft；base PR9 分支 / `8179533ffade1027d01727cbf5a89a26b491c400`；head `codex/docs-overall-status-20261011` / `0fe71a67bdc8331df9830ad69247674169b03a9d` |
| 本轮 | 从 PR10 准确 HEAD 建立独立工作树/分支 `codex/docs-overall-status-20261020`；Draft PR base 为 `codex/docs-overall-status-20261011` |

初次核对没有 PR10 之后的新文档 PR，本轮分支此前不存在。选 PR10 因其仍是最新未合并文档基线，包含 9 月 27 日需求及 10 月 11 日同步；不从 main 丢失前两轮内容，不为旧 SHA 回退任何分支。保留 main、旧工作树、PR9/10 和原有材料。界面项目虽为 FileBrowser，实际写入/提交目录已再次核实为文档独立工作树，remote 为 `Tangbohu09527/CF_ecommerce-automation-docs`。

**本轮 Draft 更新不等于 main 同步。** 后续需另行批准依次处理 PR9 → PR10 → 本轮 PR 的合并/调整 base；不预授权自动合并，不改写其他分支或 PR。

## 2. 三组件实际 PR base/head

以下均为本轮读取的完整 SHA，状态为 OPEN、未合并；不是生产版本声明。

| PR | base 分支 / SHA | head 分支 / SHA |
| --- | --- | --- |
| [Gateway19](https://github.com/Tangbohu09527/CF_agent-gateway/pull/19) | `codex/fix-inbound-session-continuation` / `f00b55681661eecbc35e95db862de84282909773` | `codex/fix-inbound-file-read-return` / `7f022fd4f28f756e1f21812ba97fc7417308281a` |
| [Gateway20](https://github.com/Tangbohu09527/CF_agent-gateway/pull/20) | `codex/fix-inbound-file-read-return` / `7f022fd4f28f756e1f21812ba97fc7417308281a` | `codex/fix-inbound-attachment-followup` / `ef3119aeecd987165255c066218441f6aef8fb0a` |
| [Gateway21](https://github.com/Tangbohu09527/CF_agent-gateway/pull/21) | `codex/fix-inbound-attachment-followup` / `ef3119aeecd987165255c066218441f6aef8fb0a` | `codex/feat-enterprise-zip-return` / `91a235fca08362d8c1440e808df56946be3cc1e8` |
| [Gateway23 原生 MCP](https://github.com/Tangbohu09527/CF_agent-gateway/pull/23) | `codex/feat-enterprise-zip-return` / `91a235fca08362d8c1440e808df56946be3cc1e8` | `codex/feat-native-hermes-remote-tools` / `1f3a1c2978bcc0b4803e6246e309fc77a7865c51` |
| [Gateway22 时区](https://github.com/Tangbohu09527/CF_agent-gateway/pull/22) | `codex/fix-inbound-attachment-followup` / `ef3119aeecd987165255c066218441f6aef8fb0a` | `codex/fix-shanghai-runtime-timezone` / `3410a50250def9d84e34099404662f5e84c8f3d3` |
| [FileBrowser1 前置](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/1) | `feat/v1-integration` / `48380c3f31cb37b01d0c05b8db0cfa49680a17f9` | `feat/file-service-clean-device` / `8ff37ef0da322aa2194d9ef37e26eb7f7035cbb2` |
| [FileBrowser2 前置](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/2) | `feat/file-service-clean-device` / `8ff37ef0da322aa2194d9ef37e26eb7f7035cbb2` | `feat/filebridge-hermes-crud` / `bb11be4944b5325f2d10aa5a1adb88a3f467da72` |
| [FileBrowser4](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/4) | `feat/filebridge-hermes-crud` / `bb11be4944b5325f2d10aa5a1adb88a3f467da72` | `codex/fix-inbound-file-read-return` / `8d12d21e59619c10a794d997edc895a912b58cb5` |
| [FileBrowser5](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/5) | `codex/fix-inbound-file-read-return` / `8d12d21e59619c10a794d997edc895a912b58cb5` | `codex/feat-product-materials` / `3d556786068994826b8105ff4a1d11572b84b1b1` |
| [FileBrowser6 时区](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/6) | `codex/fix-inbound-file-read-return` / `8d12d21e59619c10a794d997edc895a912b58cb5` | `codex/fix-shanghai-runtime-timezone` / `91ad46787bdf005ca18d599c4204a5d01f190cbb` |
| [FileBrowser7 简部署组件](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/7) | `codex/feat-product-materials` / `3d556786068994826b8105ff4a1d11572b84b1b1` | `codex/feat-unified-server-component` / `e9fc639d1916eb2d1aa61e869737bee2b38b8601` |
| [WeChat9 文件名](https://github.com/Tangbohu09527/CF_agent-wechat/pull/9) | `codex/fix-wechat-original-media` / `9b997ac92d653380a01ac93e0ac65e4a0e3a4c0a` | `codex/fix-wechat-return-filename` / `7bd42b623d405956080a13481bbf4de21aa3477a` |
| [WeChat10 时区](https://github.com/Tangbohu09527/CF_agent-wechat/pull/10) | `codex/fix-wechat-return-filename` / `7bd42b623d405956080a13481bbf4de21aa3477a` | `codex/fix-shanghai-runtime-timezone` / `e9f751819fb8e9bc1772d7f6d67089e5b49f10d7` |

读取时 Gateway main=`7963bc0db5a01e38099e021959feabf648db7ede`；FileBrowser main=`4750a97cfdf5bd067e04b6b36bf9616f5ada836d`；WeChat main=`81d21f451adc61df71077ca3035adeddeca4621f`。FileBrowser PR4 前置 PR2/PR1 功能栈未合并，详情沿用[10 月 11 日固定依赖](./2026-10-11-overall-status-evidence.md#2-三组件代码合并与-ci)。源码未合并不等于现场未曾应用其中固定产品代码。

## 3. 固定报告与实际成果

| 固定来源 | 本轮采用结论 |
| --- | --- |
| [Gateway PDF/PNG 报告 @acd756da6ff401cb1267b184faf0730d23977d21](https://github.com/Tangbohu09527/CF_agent-gateway/blob/acd756da6ff401cb1267b184faf0730d23977d21/docs/validation/2026-10-18-inbound-read-return.md) | PDF Message4310 / Dispatch49 / Delivery44：正文读取、79,083 字节原附件返回及样本摘要一致；4311/50/45 后续文字同会话接续。PNG4314/53/Attachment6/Delivery48：实际读取、准确描述和微信图片实收；历史像素一致、文件字节不同，原因未定位。按本轮用户决定：PNG图片识别及微信图片收发已验收，满足当前业务要求。 |
| [FileBrowser 商品资料报告 @3d556786068994826b8105ff4a1d11572b84b1b1](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/blob/3d556786068994826b8105ff4a1d11572b84b1b1/docs/PRODUCT_MATERIALS.md) | 普通业务账户及账户/Token 权限交集下，61 次真实 API 支撑复制/移动/改名/替换/可恢复删除/还原及回读。实际模型 Dispatch54 查看12图分类4主图/8详情，但最终导入409失败；后续 Dispatch55 新操作完成同一冻结版本，原包保留，82次独立只读API/成员复核通过。不能写每种操作均由模型选用，不能写全企业资料库完成。 |
| [Gateway ZIP 完整报告 @91a235fca08362d8c1440e808df56946be3cc1e8](https://github.com/Tangbohu09527/CF_agent-gateway/blob/91a235fca08362d8c1440e808df56946be3cc1e8/docs/validation/2026-10-18-enterprise-product-zip.md#normal-wechat-zip-outbound-acceptance--october-19-completed) | 已读到最后 `Normal WeChat ZIP outbound acceptance — October 19, completed`。10月19日10:09正式工具安装，官方认证 `/v1/toolsets` 确认六工具；18:01再次确认。正常文字 Message4320 → Dispatch59 → 新 Artifact → Delivery50，模型实际 search/list/list/export/checksum 与 fresh return，从企业文件系统导出新包，微信实收通过，不是转发刚上传原包。 |
| [clean-device @91a235fca08362d8c1440e808df56946be3cc1e8](https://github.com/Tangbohu09527/CF_agent-gateway/blob/91a235fca08362d8c1440e808df56946be3cc1e8/docs/deployment/clean-device.md) / [ZIP 开发契约](https://github.com/Tangbohu09527/CF_agent-gateway/blob/91a235fca08362d8c1440e808df56946be3cc1e8/docs/development/enterprise-product-zip.md) | 现有正式插件/客户端桥接与最终新机验收标准分开；入站/隔离替身、任务授权、READY Artifact 和实收分开，不将工具发现替代整条业务。 |
| [WeChat 文件名报告 @7bd42b623d405956080a13481bbf4de21aa3477a](https://github.com/Tangbohu09527/CF_agent-wechat/blob/7bd42b623d405956080a13481bbf4de21aa3477a/docs/validation/2026-10-18-return-filename-candidate.md) | 固定 Rust release/镜像和 Qt URI 路径隔离构建通过；唯一性放父目录、basename及UTF-8 URI候选不等于现场切换或原名实收。正式收件仍有前缀/中文替换。 |
| [WeChat 时区报告 @e9f751819fb8e9bc1772d7f6d67089e5b49f10d7](https://github.com/Tangbohu09527/CF_agent-wechat/blob/e9f751819fb8e9bc1772d7f6d67089e5b49f10d7/docs/validation/2026-10-18-shanghai-timezone.md) / [FileBrowser 时区报告 @91ad46787bdf005ca18d599c4204a5d01f190cbb](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/blob/91ad46787bdf005ca18d599c4204a5d01f190cbb/docs/SHANGHAI_TIMEZONE.md) | 源码默认值候选；自身不能证明生产切换。较晚现场报告中的时区调整和候选源码合并/镜像分别记录，不据早期“未切换”否定后续现场配置结果。 |

### 本轮新增 PR23：原生 HTTP MCP 的已证实边界

[Gateway PR23 固定报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/1f3a1c2978bcc0b4803e6246e309fc77a7865c51/docs/validation/2026-10-20-native-http-mcp.md) 的固定 HEAD 为 `1f3a1c2978bcc0b4803e6246e309fc77a7865c51`，仅新增 workflow、两个隔离探针与实验报告，没有统一服务端正式部署入口的实现，也没有生产采用。官方 Hermes `427d605d4985c2d49a795493ef6c5d66413057d1` 以新 HOME 和少量官方配置运行，未改源码，无项目插件、项目 EXE、PYTHONPATH 或处理器注入。两个实际会话并发 API200/200，合成文本/PNG 进入正式 `role=tool`；工具选择使用 HTTP 模型替身，没有真实付费模型。

[专用 Linux CI39302151607](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39302151607) 直接日志证明真实 FileBrowser `3d556786…` TLS/鉴权/HTTP fixture 的普通非管理员测试用户执行 query2/export2。合成输入18,156,370字节，输出18,156,942字节/12成员，整包和逐成员摘要一致；这不是前述真实商品微信 ZIP 实收，fixture source scope为`/`，未覆盖完整员工子目录授权矩阵。该workflow在`ubuntu-latest`运行，不是全新Debian安装/重启验收。

两路业务会话共用MCP transport session，静态连接凭据/Profile、每次`tools/call._meta`为空，缺可信Gateway task/session/employee的每调用绑定。`query_current_task`是实验拒绝哨兵，返回`trusted_task_context_missing`，不是已部署业务端点；日志为`task_return_authorized=false`。**未进入当前任务Artifact/Response发布与微信协议替身交付**，身份错配、过期、跨会话和重放完整端到端矩阵未通过。不得用模型参数、提示中的task ID或共享管理员Token代替可信绑定。统一入口与官方Hermes纯配置完整业务交付继续开发验证中。

### ZIP 成功和失败分别保存

正常出站 Artifact 为 `36860c4b-d207-5f6d-a6ce-db337e1a0441`。实际导出、存储 Artifact、收件文件均 **18,167,063 字节，12 个成员**，整包 SHA-256 为 `239fe0ddf1570553ae9fb09639ce3236f09e2fc0af843b7e1f0cc41ed48f3e42`，成员路径、大小、摘要与企业已发布选择一致。原保留包是 18,166,503 字节、SHA-256 `0e4456af887684854217dbc3d11d9104fad54dff2df82e3c41bd25e0c036c075`，重导出包的外层摘要无需等于原包。内部文件完整性与外层文件名限制分别判断。

**Message4319 / Dispatch58 / media7** 为 `timed_out / media_wait_expired`，未取得附件、未调用模型。Delivery49 发送一次失败说明；Dispatch58 的 success 仅指失败通知流水线，不是 ZIP 成功。不得改写为成功或诊断为文件损坏/格式不支持。正常出站59不覆盖 ZIP 入站缺口。

较早 Dispatch56 导出后返回被拒、Artifact0/Delivery0，以及受控 Dispatch57 只有存储 Artifact、未真实微信投递，均保留原事实；后来的59补齐正常出站实收，不重写早先结果。FileBrowser 报告10月19日10:10的“微信未测”和ZIP报告中途“未安装/未返回”仅为较早观察，不能覆盖最后完成节。

### 最后现场观察与清理时间

Gateway 报告实际观察 **2026-10-19 18:36:00 北京时间**：六个核心容器健康、原 Bot 登录、收发关闭，无非终态或 claimed Dispatch/Delivery/media、无待准入。**18:36:04.975** 撤销临时管理授权并负验证，**18:36:05.034** 删除临时加密密码并负验证。业务凭据、用户原密码和服务配置保留；不能把管理权撤销直接写成最终部署的业务连续性验收。

本轮未复核实时现场，不将观察日期改为10月20日，不沿用旧PID。现有归档路径调整由用户明确暂缓。实际原名收件未验收，文件名候选未生产切换；PNG已满足当前业务要求，不再将原始字节差异、重编码或单独file路由作为待修开发项，未定位原因不归因具体组件。

## 4. 现场产品、镜像与报告提交分列

以下均来自上述固定报告所述的现场观察，非本轮登录取得；Image ID 与 registry digest 是不同对象。

| 对象 | 报告中的实际身份 |
| --- | --- |
| Gateway 运行产品源码 | `b0517d2222fcf5a08dc6aed079ffbb8baa133203` |
| Gateway 运行 Image ID / schema | `sha256:0bbbb3c29b8117b000027adda522bdecbfb73c1756d44c08431fd1ee43fc1049` / `20261018_01` |
| FileBrowser 服务端源码 | `d505fc6599608b219f3494979b0a5f197fdf356f` |
| FileBrowser 运行 Image ID | `sha256:1e6ccdc7ff1854e1a98821a9623136315845a10e5d9a3b5cfbc2d163b4ba5fb0` |
| 正式 Windows FileBridge 插件/客户端源码 | `afcd5c875905a849a7636ccc22057b47da419dfe`，仍为项目插件与客户端桥接 |
| 官方 Hermes 源码 | `427d605d4985c2d49a795493ef6c5d66413057d1` |
| WeChat 实际运行 Image ID | `sha256:3b14e58c908ec11e9163ef0bbb51522bea15c1568c8e1012a29890b9adadc48e` |
| WeChat registry digest / 已保留溯源源码 | `ghcr.io/thisnick/agent-wechat@sha256:97d5a99f933592c15897a7579b4e2b2cbbb5f4dda856228f8271f91dd7cafb43` / `96c2e90c497d999bf5189a134a9b3b293731f42e`；不是未部署的PR9候选 |
| 总文档所引用报告提交 | Gateway ZIP `91a235fca08362d8c1440e808df56946be3cc1e8`、PDF/PNG `acd756da6ff401cb1267b184faf0730d23977d21`、FileBrowser `3d556786068994826b8105ff4a1d11572b84b1b1`；均不是“全组件当前版本” |

本轮不补造 Gateway/FileBrowser 的 registry manifest digest，也不把源码标签当成完整制品证明。Codex 开发模型改为 6.1 Sol 不表示生产 Hermes 模型、API 配置或版本改变。

## 5. 准确 SHA 的 GitHub CI

仅引用本轮读取的 run/check 结果，未在本地运行产品测试，未把历史报告自身的“CI待查”当最新结论。

| 固定 HEAD | 已核对结果 / 精确 run |
| --- | --- |
| Gateway `91a235fca08362d8c1440e808df56946be3cc1e8` | 8/8 workflow completed/success，包括 [PR CI39239457934](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39239457934)、[push CI39239449808](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39239449808)、[ZIP39239457921](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39239457921)、[PDF/PNG39239457937](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39239457937)，以及两次兼容、两次clean-device。隔离模型/微信替身不等于真实业务。 |
| Gateway `7f022fd4f28f756e1f21812ba97fc7417308281a` / `ef3119aeecd987165255c066218441f6aef8fb0a` / `3410a50250def9d84e34099404662f5e84c8f3d3` | 本轮准确HEAD查询分别7/7、7/7、6/6 workflow success，不推导候选已生产部署。 |
| Gateway `1f3a1c2978bcc0b4803e6246e309fc77a7865c51` | [原生MCP39302151607](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39302151607) completed/success；同SHA另一次MCP运行39302135529 cancelled。冻结时主CI39302151528、兼容39302151630、clean-device39302151655仍in_progress；不等待，不称全部CI绿或完整业务通过。 |
| FileBrowser `3d556786068994826b8105ff4a1d11572b84b1b1` | [常规39189416462](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/39189416462)后端测试、前端/文档等通过，**后端lint失败**（日志111 issues）；[客户端39189416508](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/39189416508) Linux客户端及官方Hermes兼容通过，**Windows客户端失败**（旧fixture HTTP410/expired）。非全绿。 |
| FileBrowser `8d12d21e59619c10a794d997edc895a912b58cb5` / `91ad46787bdf005ca18d599c4204a5d01f190cbb` | 对应测试/文档/部署等有成功；后端lint仍失败，8d12 Windows客户端亦失败，不推广为全仓成功。 |
| FileBrowser `e9fc639d1916eb2d1aa61e869737bee2b38b8601` | [部署39300420116](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/39300420116)validate-deployment通过，后端/前端及文档检查通过，后端lint失败；不代表统一Debian完整业务通过。 |
| WeChat `7bd42b623d405956080a13481bbf4de21aa3477a` | [39122067866](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39122067866)固定Rust release/Qt和Ubuntu22镜像构建通过；相关部署/Controller/隔离媒体检查通过，不代替现场原名实收。 |
| WeChat `e9f751819fb8e9bc1772d7f6d67089e5b49f10d7` | [39123324561](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39123324561)fixed-runtime及相关身份/隔离媒体/部署检查通过，不代替现场镜像切换。 |
| 总文档 | Workflow查询total_count=0，PR10准确HEAD check-runs为空，**无文档CI**。本轮检查结果记录于自己的Draft PR；不得写CI全绿。 |

## 6. 并行开发、缺证与下一步

- Gateway统一服务端入口、官方Hermes纯配置接入由另一个任务负责，开发验证中；最后冻结查询发现已正式推送的PR23，已按固定HEAD补入原生MCP边界；它不含统一服务端正式入口实现，不根据本地未提交材料猜测。正式推送的FileBrowser PR7已作为必要增量：固定源码、时区合入及首次准备中断后的受保护意图恢复/拒绝有候选验证，但没有生产部署或真实微信操作；其明确的Debian/集成边界保留。
- MCP仅验证路线，PR23隔离发现/图文/导出已证实，但可信每任务绑定及Artifact/Response/微信整链未打通，不等于完整业务通过。最终目标、当前插件桥接和逐项验收见[部署指南](../../deployment/deployment-guide.md#最终部署标准与候选边界)。本任务不等待并行工作完成，不修改其分支或PR。
- 已发布GitHub脱敏报告可读取；私人原始日志、生产、实际新候选镜像切换、ZIP完整入站、文件名候选实收、真实附件引用归一化/显式续办及最终新机业务连续性没有本轮独立证据。缺证不向用户索要凭据或生产权限。
- 资料整理与首批Skill可小批量安排，不等ERP、全部部署、新RAG或自研Runtime；已有工具/缺业务规则/缺现场验收见[当前进度](../../status/current-progress.md)。这不是已交付或工期承诺，归档路径调整仍暂缓。
- 项目最后三项、PNG业务验收取舍和部署目标已记入[技术决策](../../05_技术决策记录.md)，原六阶段、历史日期/失败、唯一File Service、权限/审计与READY Artifact边界保持。可靠性、ERP/平台等无新增证据的部分不扩大完成声明。
