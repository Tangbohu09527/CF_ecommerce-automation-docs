# 2026-10-20 统一部署与官方Hermes纯配置候选证据

## 本轮目标与操作边界

本轮实际开发第一版候选，包含Gateway编排的统一Debian入口、固定官方Hermes原生HTTP MCP验证，以及FileBrowser/WeChat必要的所属仓最小适配。开发模型按用户选择6.1 Sol。最多两条工作线，同一分支单写入者；文档在关键结果后收尾。

没有连接生产服务器、生产API、正式Hermes或真实微信；没有读取生产凭据、企业素材或正式业务目录；没有登录、扫码、UAC、收发开窗、迁移生产、重建生产镜像或改正式配置。没有创建/恢复运维SSH/sudo授权。本轮CI临时数据、证书、用户和密码只属于一次性隔离环境，不是生产管理窗口。

通过现有已认证gh API按仓库全名读取默认分支、PR base/head/状态、固定源码、实际diff及Actions；没有用搜索摘要或不带ref的默认文件代替候选。四仓独立分支，原工作树/未提交材料保留；不merge、force push、发布正式版本。文档仓只修改Markdown。

## 本轮结论

| 交付 | 实际结果 |
| --- | --- |
| 统一服务器入口 | 候选已实现；精确HEAD b3f2740的隔离Debian容器完整检查通过，18,168,526字节/12成员、失败接续、撤销管理密码后业务Token仍可用、服务重建及第二次执行均实测 |
| 官方Hermes纯配置 | 官方依赖环境、原生图文、真实FileBrowser查询/导出及约18MB完整性通过；可信每任务上下文仍缺，未进入Gateway当前任务返回、Artifact/Response和微信替身 |
| 四仓与正式发布 | 独立候选及文档已普通提交/推送，均保留Draft；没有合并或正式发布。准确SHA/CI及边界见下文 |
| 最终新机标准 | 尚未整体达到；新增统一入口仍缺booted-systemd三组件/持久宿主盘/整机重启，新Windows及跨机器MCP HTTPS也未证明 |
| 生产 | 完全未访问、未改动；现有PDF/PNG/商品ZIP出站的历史成功保留，不能据此声明此刻在线 |

两个隔离组合各自验证，未拼成已经通过的纯配置业务链。下面的统一入口18MB测试是文件服务真实上传/下载；原生Hermes组合另有真实产品export，但没有当前任务合法返回闭环。

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
| 统一服务器候选 | PR24，当前候选 `b3f2740289c6b1fa3208c352bc87b99b6f50cce4`；旧尝试另列 | PR7 `47876e3a3d62affe777de8623697f62791fa5fa4`，基于PR5并集成PR6、准备恢复和公开镜像资产读取修正 | 管理PR11 `e56c7e4b606f1614e54210a391906f4be7440f5e`；运行镜像为下列固定upstream digest；Hermes仅配置URL、不调用 |
| 原生Hermes探针 | PR23，官方依赖验证代码 `eac246359e8baa7fe97c253cd13cd9f0667949b0`；旧子集结果另列 | 固定PR5 `3d556786068994826b8105ff4a1d11572b84b1b1` 的真实HTTP fixture，不是统一安装容器 | 官方 `427d605d4985c2d49a795493ef6c5d66413057d1`，官方锁定Python3.14.7及默认完整[all]；微信未接入，未用生产模型Key |

统一入口选择的协议镜像为 `ghcr.io/thisnick/agent-wechat@sha256:97d5a99f933592c15897a7579b4e2b2cbbb5f4dda856228f8271f91dd7cafb43`；其既有来源观察为协议源码 `96c2e90c497d999bf5189a134a9b3b293731f42e`。管理仓源码含PR9不等于该镜像启用了文件名候选；本轮不切换协议。组件manifest不把不同分支的“各自最新”拼在一起。

## 实际候选变更

- Gateway PR24：`deploy/install-server.sh/.py`、固定组件manifest、受保护输入模板、HTTPS接线、阶段journal和隔离CI。调用三个现有入口，不复制FileBrowser业务。继承PR22的Asia/Shanghai默认，SQL语义UTC；原Gateway执行器只新增可选root-only诊断日志。
- FileBrowser PR7：集成PR6时区默认；准备阶段在任何正式目录写入前保存受保护intent，原随机凭据和文件字节固定；中断后只补缺失且一致的文件，拒绝不明半成品、并发变更和输入漂移，完成后删除intent。新增5项恢复测试。未改产品业务API实现。
- WeChat PR11：现有prepare增加仅root可用的root-managed checkout/configure；不要求新建锁定manager再次输入密码。保留固定来源、路径/身份验证；login venv延后到既有登录入口。未登录、创建runtime、授予sudo或开闸。
- 总文档：更新当前状态/进度、部署导航、需求的后续状态、总纲当前摘要及本记录；六阶段原定义不变，无运行代码或真实企业资料。

原有生产插件/两个EXE未修改。当前机器修通不等于最终“官方Hermes加少量配置”交付。本轮不把新客户端藏进脚本作为替代。

所属仓文件与原因可按固定PR diff核对：FileBrowser的`deploy/shared-host/lifecycle.py`实现准备恢复，`deploy/tests/test_shared_host_prepare_resume.py`验证文件/凭据保留与拒绝边界，`deploy/shared-host/README.md`说明恢复契约，`.github/workflows/deployment-assets.yaml`接入检查；两个`_docker/Dockerfile*`、`deploy/compose.yaml`和`deploy/shared-host/compose.yaml`设上海日志默认，`deploy/systemd/filebrowser-enterprise-backup.timer`明确计划时区，`deploy/shared-host/validate.sh`及`scripts/lib/deployment_validation.py`对齐该契约，`deploy/tests/test_timezone_defaults.py`与`docs/SHANGHAI_TIMEZONE.md`分别测试/说明。追加`deploy/shared-host/image.py`验证公开资产在普通10001:10001下可读；镜像仅公开dist文件/CSS为0644、目录0755，不扩大数据/凭据权限。合计14文件、327增/22删，后端tree未改，工作树干净。

WeChat只改4文件、115增/11删：`scripts/prepare-clean-host.sh`实现受限root-managed准备，`tests/unit/test_root_managed_prepare.py`测边界，`.github/workflows/unified-component.yml`执行相关检查，`docs/deployment/clean-device.md`说明入口及不登录边界。工作树干净。Gateway入口/manifest/模板/HTTPS与journal归本仓；原生探针、官方依赖安装/审计、workflow和报告留独立PR23。文档仓9份Markdown只做README/AGENTS导航、状态和证据收尾。

实际验证命令包括Gateway的`python3 -m unittest discover -s tests -p test_unified_server_entry.py -v`、`bash tests/deployment/run_unified_server.sh`，FileBrowser的`python3 -m unittest discover -s deploy/tests -p test_shared_host_prepare_resume.py -v`、`python3 -m unittest discover -s deploy/tests -p test_timezone_defaults.py -v`和正常`make validate-deployment`，WeChat的`bash -n scripts/prepare-clean-host.sh`及`python3 -m unittest discover -s tests/unit -p test_root_managed_prepare.py -v`。完整流水线和实际结果按下文准确run区分；未重跑无关全仓矩阵补“绿色”。

## 官方Hermes原生实测

先前成功探针的适用范围在依赖审计后收窄：旧CI39302151607/39302780142安装了自选pip子集，没有使用官方完整锁定环境；PyJWT、httpx2、starlette实际版本分别偏离官方pin，多项core包缺失。Windows Python3.12也只是非官方支持运行时诊断。以下旧结果保留历史，不用它们证明完整官方安装。

修正代码 `eac246359e8baa7fe97c253cd13cd9f0667949b0` 的[CI39315976156](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39315976156)已成功；root独立下载并读取`dependency-summary.json`与`safe-summary.json`。实际调用固定官方`scripts/install.sh --stage python-deps`，由PM按官方锁安装默认完整`[all]`，解释器为锁定Python3.14.7+20260901。53项适用core/extra约束全部满足、missing/mismatches均为空；PyJWT2.13.0、httpx2 2.7.0、starlette1.3.1，源码工作树干净。只退出官方允许的可选浏览器/CUA二进制，不删Python依赖；这是实际官方依赖阶段，不冒称完整安装器所有阶段或新电脑验收。

本次新结果：两路API为200，模型替身实际选择8次工具、14次模型循环，无模型错误；原生图文进入正式tool消息，普通FileBrowser测试用户真实TLS查询/导出各2次。输入18,156,370字节、SHA256 `2f63163f6265c9d7026f8fd5793ad6e099182eedb42ecede55bc9a0237e90a74`；实际导出18,156,942字节、SHA256 `c37cd162e6ec1432a843891685c544a8ada64760cf55e1319a711fc74fa4bb4d`，12成员逐项匹配。项目插件列表为空、子进程没有PYTHONPATH。所有8次调用的`_meta={}`、两任务调用共用一个transport，`trusted_task_context_missing`仍被实际拒绝，`task_return_authorized=false`。因此严格官方依赖环境没有消除可信任务关联缺口，Artifact/Response/微信替身仍未到达。

测试分层：官方API、MCP和模型替身位于同一隔离runner、使用loopback HTTP；真实TLS指MCP服务到FileBrowser fixture这一段。没有把该探针当作Windows新电脑、跨机器MCP HTTPS/TLS配置或真实Debian文件服务部署。正式处理器、数据库、文件字节和官方调用真实执行，模型与微信边界按上述替身/未到达标注。

官方锁定输入摘要分别为`pyproject.toml`：`62541c282d5d4b1c4b43c02f3072477597ef024b28a3d1bd3acadd29934f050d`，`uv.lock`：`7f337b67943cbb287651761c8b40a2d8e1e595b5aff47dd7deb2683717c3be41`，`pm/lock.json`：`49b79259220160f147255078a19af541a5f328d3871c8041cc1eb909d19eb549`。以上为脱敏实际事实，合成Token、完整配置、ZIP和私有日志未上传。

当前[固定详细报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/57bb2c2830f292d5167f0bda449afafbe0a528fd/docs/validation/2026-10-20-native-http-mcp.md)保留新旧环境的区别。39315976156的PR合成checkout为`cf46b38970e05b6c58b7e263df9e92afee0d0d8b`，与代码eac2463的tree均为`b3903ee0e14a5636779dd1fdebb0c001e00113df`；后续57bb2c2只改报告。其准确最终HEAD检查另列，不把旧代码检查当成整个新HEAD结果。

最终文档HEAD `57bb2c2830f292d5167f0bda449afafbe0a528fd` 的[原生CI39317205768](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39317205768)于10月20日04:12:58北京时间通过。root再次独立读取两份JSON并用GitHub Git API核对：实际checkout `99ef201a46793d9d414337c580b3aa18526aadb0` 和最终HEAD的tree均为`446c6f2ce2026bad7b28591e5fc6068ac956d062`。此轮输入SHA为`0d232ceb201f79ec14f3c2243dd8e5d0700d3c10045b8ef52bf36d3871f10e8b`，导出/12成员与上次一致；官方依赖、图文、无插件/PYTHONPATH及可信任务拒绝均再次实际核对。绿色结论是边界探针成功，完整业务返回仍未通过。

### 保留的旧子集与Windows诊断

[旧子集报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/2ada096c8a30ecbde771fb32f7536790709b0cd9/docs/validation/2026-10-20-native-http-mcp.md)、[旧代码CI39302151607](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39302151607)、[旧HEAD CI39302780142](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39302780142)均保留其限定结果。root曾核对后一safe-summary输入摘要`8b6e2b837740371b0e2e22f2549ba6947b0e92606c114d362f6585474130de49`，导出与12成员匹配。以下旧环境数据不是正式依赖验收：

干净官方HOME启动前仅写入配置与合成凭据，官方运行时会生成缓存和会话等文件；无项目插件/EXE、PYTHONPATH、源码改动或启动注入。真实官方API两会话创建201、两路请求200；确定性模型替身实际选择原生远程工具，图文进入正式role=tool。FileBrowser真实TLS/HTTP处理器、临时数据库和普通用户执行query/export各两次；fixture source scope为`/`，不能冒称已验更细员工目录矩阵。

代码CI该次输入18,156,370字节，SHA256 `12a212577a83c38bf410e3512036da61955f1af66afe9e0e114f882493db5f29`；导出18,156,942字节，SHA256 `c37cd162e6ec1432a843891685c544a8ada64760cf55e1319a711fc74fa4bb4d`。实际TLS下载字节与导出回执一致，12成员大小/摘要与选定输入一致。重新打包不要求外层ZIP与入站容器相同。

**确切缺口：** 两个会话共享同一MCP transport，8次tools/call的`_meta`均为空。固定官方handler调用`session.call_tool(tool_name, arguments=args)`，未转交API当前任务/会话上下文；identity_header只支持静态或连接时Profile。服务端诊断哨兵实际返回`trusted_task_context_missing`。该哨兵不是已部署Gateway业务端点。

因此尚未进入真实Gateway当前任务的合法返回授权、Artifact/Response和微信替身投递；错身份、过期、跨会话、重复使用的完整端到端矩阵未完成。没有让模型填taskID，没有用共享Admin或移除鉴权。单任务专属受限grant/官方配置进程是最小替代候选，但会新增可信任务生命周期管理；本轮未偷偷实现或当作已满足要求。

最少原生配置形状（仅证明通道，不是可部署完整业务）为官方config中的`mcp_servers.cf.url`、静态受限`headers.Authorization`、`identity_header: {name: X-Hermes-Profile, value_from: profile}`、`platform_toolsets.api_server: [mcp-cf]`，加既有官方模型/API配置。只有profile或静态header不能区分同Profile并发任务。

初次Windows图片写入PermissionError保留；正常权限隔离进程重跑成功，未改ACL。Linux探针曾误识别工具描述/不可信内容包装，修正替身解析真实回执后通过，未删除完整性断言。成功是明确的边界探针成功，不能表述为全业务通过。

## 统一入口验证与保留的失败

本地首轮4项输入/journal检查、追加中断镜像回归后共5项检查、Ruff和diff检查通过。PR24专用CI在一次性Debian13容器中调用统一入口，独立真实Docker daemon，不挂host socket；四项systemd只读观察为替身。测试使用独立tmpfs挂载test-disk，不是实际RAID或持久宿主磁盘。未运行booted systemd或整机重启；旧clean-device工作流的单组件A/B不能扩大为新增统一入口B通过。

首次检查 `39303826610` 在 `297729718411886405764e794cb5bc5a8bf46e19` 失败，已下载脱敏result.json及Debian digest。system、依赖、Controller、WeChat checkout/configure/bootstrap、独立凭据、FileBrowser checkout/image实际完成。FileBrowser准备拒绝test harness用默认权限挂出的tmpfs：`unsafe installation parent: /srv/storage`。这不是预定端口冲突，不能算其通过；重试后仍拒绝且没有继续start。

修正提交 `7e3321ad67bee523d52e4df6fdf65617fe7b3550` 仅改测试：tmpfs显式mode0755、预期失败必须实际诊断为port phase，并增加镜像阶段完成记录已写/journal未提交中断的本地回归（5项全部通过）。产品实现未放宽权限，旧失败保留。该次新检查`39305292066`在FileBrowser的lan-inputs阶段正确拒绝测试输入中的loopback CIDR，预期port断言也准确报红；并未继续启动服务。其脱敏错误位置为validator shell line247。

修正`71c802da233cfd855d0b7436da543c823a937098`去除测试loopback CIDR，并在统一输入层提前对齐FileBrowser的规范化unicast CIDR、去重和DNS label契约，本地5项检查通过；还将固定source/image公共回执以严格字段白名单加入脱敏结果，自动push/PR按同分支去重。后续实际结果分项记录如下，不把取消/未完成检查算通过。

该提交的统一入口`39306797027` attempt2已实际证明预定端口拒绝及移除故障后的prepare接续，随后FileBrowser start的Docker命令失败。最初输出被安全抑制，未假定根因。诊断提交`e75705eeeda7291ba4d158da3131442088ea019f`仅增加固定项目容器状态、有界daemon错误和生成凭据脱敏；检查`39309184393`取得明确错误：cgroup v2 `/docker`父组处于threaded mode，无法应用FileBrowser未变的资源限制。

测试修正`0d057c8c7f717e240ab0ad7f5fab8a7df37bf803`只在显式private cgroup namespace内做控制器委派，并在应用构建前运行CPU/内存/pids限额的smoke；沿用[固定Moby DinD实现依据](https://github.com/moby/moby/blob/4eee42c0243b1494b5df6411c1c208f7408461d8/hack/dind)，改为有限重试。不共享host Docker socket或cgroup namespace、不去掉产品资源限制、不改鉴权或TLS。

该版`39311006559`实际通过资源smoke，OCI错误消失，随后FileBrowser重启/退出126。固定入口的Git checkout继承全局077 umask，使被bind到容器的已审核入口脚本为root-only 0700。修正`030ee8d09e6a0362b624cd5e7b1032895e825138`只对该固定普通代码文件验证身份/链接/模式并设0755；不递归放宽源码、不改任何凭据、owner或服务10001身份。新增POSIX回归验证仅目标改变、私有文件不变、不明模式拒绝；本机5通过/1项POSIX跳过，Linux须实际执行。`da369b579716c1dc825d729829c9f172e417df46`再将FileBrowser固定构建的base digest及源码依赖输入摘要以白名单导出，未上传完整private日志。最终准确检查待执行结果补齐。

## 组件准确CI与已知红项

统一检查39313438306（da369b5）已越过入口126：FileBrowser start/bootstrap-finish、Gateway build/configure/database/migrate/start完成；HTTPS start unhealthy。本次还发现普通10001读公开CSS/favicon失败。FileBrowser提交47876e3只规范公开镜像资产读取并实际用10001无网络/挂载只读容器检查。其13项image-workflow测试本地通过；部署资产CI39316167228通过。Gateway提交0451619固定该来源并补HTTPS固定项目Health/有界启动错误。

实际检查39316302951（0451619）中FileBrowser健康、上述权限错误列表为空；HTTPS反复退出1，Nginx日志明确`open() "/etc/cf-https/nginx.conf" failed (13: Permission denied)`。原因是HTTPS目录请求0750受077 umask变为0700，服务组不能遍历。提交89c1f04显式只将该root:10001只读挂载目录设0750；父state仍0700、证书/私钥/配置仍0440，不改TLS和普通服务身份。新增真实POSIX回归，本机5通过/2项POSIX明确跳过，由Linux CI实际执行。隔离管理检查也补上真实密码sudo正验证、清ticket、锁定、同密码与sudo负验证，再读独立业务文件；该完整运行结果按新SHA另列。

[39318893826](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39318893826)（89c1f04）已完成全部记录的安装阶段，HTTPS/FileBrowser均健康，HTTPS遍历修正实际奏效；随后基础状态调用完整Gateway `diagnose`退出，尚未进入18MB、撤权、重建和重复执行检查。脱敏摘录只显示空业务状态及`command_failed:docker`，没有内层诊断JSON，因此不编造具体网络错误码。源码/输入跟踪显示该诊断必然尝试外部Hermes连接，而本轮基础安装刻意使用不可用的AI `.test`地址。修正`5f3b63f167ecac825e014775aba2d0b925d1d19e`新增`diagnose-base`，复用数据库、核心健康及当前业务配置检查；原`diagnose`保留外部连接行为。先观察到边界回归失败，再修正；本地6通过、2项POSIX明确跳过，Ruff/格式/空白通过。初次受限Windows临时目录PermissionError单独保留，正常执行身份重跑通过，没有修改ACL或绕过产品权限。Linux中的核心故障传播与真实统一入口仍由新SHA CI核验。

[39321391785](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39321391785)（5f3b63f）实际通过基础诊断，完成严格HTTPS整包上传/下载和12成员摘要；随后对穿越路径返回400，与固定FileBrowser `backend/http/resource.go`的无效路径拒绝契约一致。测试原先只接受401/403/404导致检查退出，不能据此说服务接纳了越界路径，也不能把该失败运行写成后续撤权/重建/重复执行已过。测试修正`b3f2740289c6b1fa3208c352bc87b99b6f50cce4`仅要求该路径精确400，另用普通Token请求真实存在的Admin用户记录要求403，并保留错Token拒绝；没有改产品权限或删除断言。传输完成事实提前写入脱敏checkpoint，防止后续独立失败掩盖已完成阶段。Ruff/格式/空白通过，整体结果按该新SHA记录。

单组件clean-device检查`39306797023`（`71c802da233cfd855d0b7436da543c823a937098`，attempt2）A/B均已成功。root已下载B-result.json：真实Debian13 VM、PID1 systemd、KVM、Docker/PostgreSQL/Gateway及两次boot ID变化；配置/测试凭据和已登记授权保留，收发保持关闭。Hermes仅TCP替身、微信为auth-only及内部入站替身，未调用模型/真实微信。此检查仍使用原WeChat管理`67cbbb04ce15703428ce165ac38effac19f4b701`和镜像digest`b5e92047e28ce67e34576e574d8ccf00f8619f485597109f7342a137300285c0`，不含FileBrowser；不能把它写成PR24新增三组件统一入口的booted-systemd验收。

上一统一候选HEAD `89c1f046099fed77c9cffea7c8efbbd724c99270` 的[既有clean-device39318893786](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39318893786)也已通过。root下载并读取其A/B结果：B仍为上述独立旧WeChat组合、真实systemd/KVM及两次boot ID变化；A包括真实PostgreSQL重启保留数据、Hermes HTTP和WeChat协议替身。它不含FileBrowser，也未调用新增统一入口；A中合成测试短暂开关隔离Gate不属于本轮生产操作。此精确HEAD结果不扩大统一候选的证据层级。

- FileBrowser PR7：deployment-assets `39300420116`通过，涵盖新增准备恢复；本地5项恢复单测通过。regular tests `39300420042`中backend/frontend/format等通过，lint-backend失败111项。产品backend树本轮未改；既存lint不宣称修复。旧FileBridge fixture HTTP410边界保留，未把未触发/未重跑的client矩阵写成新SHA全绿。

最终FileBrowser SHA47876e3：deployment-assets39316167228通过；regular39316167220的后端/前端测试、格式、前端lint、docs及翻译检查通过，后端lint仍111项，已读实际失败日志。本轮没有为旧lint做无关重构。

统一候选实际构建并运行的是完整Dockerfile。slim镜像包含同类公开资产模式修正，但没有单独进行完整镜像构建验收；不能由完整镜像结果推定slim已通过。

FileBrowser基线PR5与PR7的`backend` tree经只读Git核对均为`349b6d58606d761703e790f97a244b4c7ce37564`；该比较只证明本轮没有改后端，不能把lint红项改写为通过。
- WeChat PR11固定HEAD：component boundary `39300943397`、Debian clean `39300943037`、forced QR `39300943020`、upstream media `39300942997`均通过；无真实微信登录验收。
- Gateway PR21固定产品SHA原8项检查通过，PR22原6项通过。它们不是本轮新入口HEAD CI；本轮准确结果待统一检查后补齐。正常自动push/PR重复及已被新SHA取代的CI允许取消，取消不算通过。
- 总文档无workflow；只做Markdown/链接/敏感内容/空白检查。

PR23最终HEAD `57bb2c2830f292d5167f0bda449afafbe0a528fd` 的正常[CI39317205893](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39317205893)三项job、[host39317205780](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39317205780)七项job、[clean39317205809](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39317205809)的A/B均成功，root已再次通过GitHub核对准确HEAD与结论。host工作流的旧探针仍为依赖子集，不替代前文官方PM完整默认依赖证据；clean是既有部署工作流，不是PR24统一三组件入口或原生授权返回通过。无须手工重复该矩阵。


## 候选清单与准确检查索引

| 仓库/分支 | 固定HEAD与base | 实际改动/检查边界 |
| --- | --- | --- |
| Gateway PR24 `codex/feat-unified-server-entry` | `b3f2740289c6b1fa3208c352bc87b99b6f50cce4`；base PR21 `91a235fca08362d8c1440e808df56946be3cc1e8` | 27文件、2381增/12删；统一入口、接线、默认时区及相关验证，不包含旧插件隐藏安装 |
| Gateway PR23 `codex/feat-native-hermes-remote-tools` | `57bb2c2830f292d5167f0bda449afafbe0a528fd`；相同产品base | 6文件、967增；官方安装/依赖审计和真实原生探针只在隔离tests/workflow，不修改官方客户端 |
| FileBrowser PR7 `codex/feat-unified-server-component` | `47876e3a3d62affe777de8623697f62791fa5fa4`；base PR5 `3d556786068994826b8105ff4a1d11572b84b1b1` | 14文件、327增/22删；所属仓准备恢复、镜像资产及默认时区；backend树未改 |
| WeChat PR11 `codex/feat-unified-server-component` | `e56c7e4b606f1614e54210a391906f4be7440f5e`；base PR10 `e9f751819fb8e9bc1772d7f6d67089e5b49f10d7` | 4文件、115增/11删；现存prepare的受限root-managed模式、测试、CI和说明；没有协议运行时切换 |
| 总文档 PR12 `codex/docs-config-only-delivery` | base PR10 `0fe71a67bdc8331df9830ad69247674169b03a9d`；本记录固定URL的commit即文档版本 | 共9份Markdown；总纲/导航/状态/部署/需求/证据，六阶段不变。文档提交不代表产品或部署SHA |

Gateway b3f2740的[统一入口39323622360](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39323622360)、[host39323622144](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39323622144)、[既有clean-device39323622091](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39323622091)均成功。后两者沿用各自旧单组件/探针组合，不替代新增统一入口的booted-host或官方PM完整依赖检查。统一入口Linux针对性8项均执行通过，本机6通过/2项POSIX跳过。[常规CI39323621819](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39323621819)最终也成功，root于10月19日21:16:50UTC／10月20日05:16:50北京核对：Python/迁移/V2、质量/仓库检查、真实Compose容器E2E三项job均成功，head为准确b3f2740。没有用旧提交绿色替代新HEAD。

FileBrowser最终部署资产[39316167228](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/39316167228)成功，常规[39316167220](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/actions/runs/39316167220)失败范围仅为前述111项backend lint；不能写全仓绿。WeChat最终[component39300943397](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39300943397)、[Debian39300943037](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39300943037)、[forced-QR39300943020](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39300943020)、[media39300942997](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39300942997)均成功。PR23最终原生/常规/host/clean成功链接与依赖分层见前文。

早期曾手工取消重复检查并出现竞态，取消记录保留；后续改由同分支concurrency自动去重，不再手工抢取消。最终自动push/PR重复运行被取消的记录不作为成功或产品故障；没有手工重复无关全矩阵。文档仓没有workflow，只有已执行的9份Markdown UTF-8/结构/相对链接/敏感模式/空白检查；不伪造CI。

审阅入口：[Gateway PR24](https://github.com/Tangbohu09527/CF_agent-gateway/pull/24)、[原生Hermes PR23](https://github.com/Tangbohu09527/CF_agent-gateway/pull/23)、[FileBrowser PR7](https://github.com/Tangbohu09527/CF_filebrowser-enterprise/pull/7)、[WeChat PR11](https://github.com/Tangbohu09527/CF_agent-wechat/pull/11)、[总文档 PR12](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/12)。均保持OPEN/Draft，不merge、force push或发布正式版本。

## 最终统一入口脱敏回执

精确源码 `b3f2740289c6b1fa3208c352bc87b99b6f50cce4` 的[39323622360](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39323622360)成功，root下载并独立读取result、Debian镜像及限定日志。工作流run_started_at为10月19日20:57:32UTC／10月20日04:57:32北京，实际隔离job于20:59:06UTC启动；成功后的updated_at为21:09:25UTC／05:09:25北京。以下白名单摘录直接来自该次JSON，不仅保留一个本机路径或摘要。原始Artifact有14天留存，长期事实以本固定文档、对应源码和CI为准。

19个实际阶段完成；预设端口冲突先准确拒绝，再接续。普通10001:10001服务及受限业务Token真实传输18,168,526字节合成ZIP，12个成员逐一匹配。管理密码经stdin设置，sudo实际正验证后清ticket、锁定；同密码及无密码sudo均负验证，业务Token仍读取原字节。此项是一次性Linux管理密码/sudo撤销，**不是SSH连接/密钥撤销或生产机器人验收**。

同镜像真实重建FileBrowser服务后文件仍在；第二次完整入口保持凭据、配置及文件摘要。存储仍是独立tmpfs测试盘，没有证明宿主盘重启持久性或恢复备份。PostgreSQL实际SHOW为SQL UTC、log_timezone Asia/Shanghai；其他组件的TZ默认配置按固定源码核对，不把未启动的微信日志说成已实测上海时间。

下面的Image ID是此轮本地构建/运行结果，registry digest与多架构Image ID分别记录。FileBrowser为完整Dockerfile的staging构建，未发布生产镜像；slim未单独构建。微信基础安装真实执行，但没有runtime/登录/真实收发。四项systemd观察为替身，不能将旧单组件A/B扩大成新增统一三组件booted-host通过。EXIT脚本执行一次性容器及私钥fixture清理，但没有导出逐项删除后的独立负验证；runner不作为长期凭据存储。没有创建生产临时授权或需要远程撤销的凭据。

```json
{
  "status": "passed",
  "scope": "pristine-Debian13-container-independent-Docker-daemon",
  "systemd": "four-read-only-observation-substitute",
  "storage": "independent tmpfs test-disk; not persistent host disk/RAID",
  "cgroup": "private namespace; cpu/memory/pids resource-limited smoke passed",
  "bytes": 18168526,
  "members": 12,
  "sha256": "5f30325a20bf69ab8575f08ec50d5fea95a00b2ef200cf7749dbf8645b9986f1",
  "filebrowser_runtime_user": "10001:10001",
  "reviewed_entrypoint_mode": "0755",
  "passed": [
    "same-input failure resume",
    "verified HTTPS upload/download",
    "whole ZIP and 12 member hashes",
    "ordinary scoped business credential",
    "management password sudo positive, locked, same password and sudo negative",
    "business credential still works",
    "real same-image service recreation retains file",
    "malformed path 400, wrong token denied, other existing user 403",
    "Poll/Delivery closed",
    "no WeChat runtime/login",
    "SQL UTC and log Asia/Shanghai",
    "second complete entry preserves credentials, config, and file bytes"
  ],
  "not_tested": [
    "booted systemd",
    "host reboot",
    "real Hermes model",
    "real WeChat",
    "configuration-only trusted task return",
    "published production image combination"
  ],
  "completed_stages": [
    "system",
    "filebrowser-dependencies",
    "gateway-controller",
    "wechat-checkout",
    "wechat-configure",
    "wechat-bootstrap-no-login",
    "credentials",
    "filebrowser-checkout",
    "filebrowser-image",
    "filebrowser-prepare",
    "filebrowser-start",
    "filebrowser-bootstrap-finish",
    "gateway-build",
    "gateway-configure",
    "gateway-database",
    "gateway-migrate",
    "gateway-start",
    "https-prepare",
    "https-start"
  ],
  "image_receipts": {
    "filebrowser": {
      "image_id": "sha256:4433479d35ca5a4cab0c4e7f7784a614f35926bcaac514314e12d4f8baebedd4",
      "reference": "cf-filebrowser-unified:47876e3a3d62affe777de8623697f62791fa5fa4",
      "source_sha": "47876e3a3d62affe777de8623697f62791fa5fa4"
    },
    "gateway": {
      "base_digest": "python@sha256:34386ef0cb081344d7ec1c103ba398e6e9f64e9ab3a1509accc92a4e24a07258",
      "gateway_commit": "b3f2740289c6b1fa3208c352bc87b99b6f50cce4",
      "image_id": "sha256:27d00f10f101d6dd3b0d6c97902e1af21286a281659338a20e0a5c34a075afb3"
    },
    "https": {
      "digest_reference": "nginx@sha256:a8b39bd9cf0f83869a2162827a0caf6137ddf759d50a171451b335cecc87d236",
      "image_id": "sha256:dc73b49f5124cf2ee538dfbdfbd121f0b4ccdcb20fea30f3a81bd477c02e2bb5",
      "requested_reference": "nginx:1.28-alpine"
    },
    "postgres": {
      "digest_reference": "postgres@sha256:0ea6700a3b4f0ae6ce746519073558aed4d88a79d8d07622a9a644946c7319c4",
      "image_id": "sha256:e8538db784b854a9baac5c1be6d059dcf0db257af2d94c8c5807b5fc15f11222",
      "requested_reference": "postgres:16-bookworm"
    },
    "python": {
      "digest_reference": "python@sha256:34386ef0cb081344d7ec1c103ba398e6e9f64e9ab3a1509accc92a4e24a07258",
      "image_id": "sha256:4f228bc1cbcfc794e4878312cfc241d368eff55a21c08d1e2bb263116c4ef524",
      "requested_reference": "python:3.12-slim-bookworm"
    }
  },
  "filebrowser_build": {
    "bases": {
      "FFMPEG_IMAGE": {
        "digest": "gtstef/ffmpeg@sha256:2e4a1c2263d8f47812d1832d9b367acf69ce89fe4e69af17757bfb6d4abfb483",
        "image_id": "sha256:e77d6b7d5c4e13c9ec2597afc1002e1a0c69438c12fa94672107d83a53445f37",
        "requested": "gtstef/ffmpeg:8.1-decode"
      },
      "GO_IMAGE": {
        "digest": "golang@sha256:f92b6ef800e499660581efdabdf25d9d817a9d124eaf900924f0504e7e27e12d",
        "image_id": "sha256:0a761904b78c29eba4ec83bd83b7312db1e113dcd92da7fc669843ad9bca56a1",
        "requested": "golang:alpine"
      },
      "NODE_IMAGE": {
        "digest": "node@sha256:c3de60bf2f9dd0ac6370e6117950ff62d6e339527e7472301c9c78a017978392",
        "image_id": "sha256:d0a16be756b6ec6658aecb8b7bcf87fa6c7d0d7fcc6ca53ad14bb49a4c90bdd7",
        "requested": "node:jod-slim"
      },
      "RUNTIME_IMAGE": {
        "digest": "alpine@sha256:294b683cb724975bec92580e1e685676bd4b50bda910ddb8c51d4cabeaec77e6",
        "image_id": "sha256:320994c3b997e2ec6433f717f153e108023c5bec8fefa8d76b83451d16d05ea8",
        "requested": "alpine:latest"
      }
    },
    "image_id": "sha256:4433479d35ca5a4cab0c4e7f7784a614f35926bcaac514314e12d4f8baebedd4",
    "inputs": {
      "_docker/Dockerfile": "6d2ecca8a8e69fd0c085c94c0d98199b269a4a14c348feb89f064f736378f6b5",
      "backend/go.mod": "906731ccff310099e4b892046148156b2bb2a4ce655aba4eb90ea4870072dbb9",
      "backend/go.sum": "fbbfda03caa850b306f7aa094ae7e0648188f2cb8b2a9f9ccb7d724ba85f7b7f",
      "deploy/shared-host/compose.build.yaml": "76732e9489872f16fae1bb4463cf6699114fc400c83867768d6de4357756b0d3",
      "frontend/package-lock.json": "29b8ca5c29a9e35114bad4555c1e4c0f9b1dd59de8e58f6430f24abc2553885a",
      "frontend/package.json": "ce5fa91cbc643eb174d642d6ebaaaebfdac22e3ceffbbdfbb7aed306e916e82e"
    },
    "platform": "linux/amd64",
    "source_sha": "47876e3a3d62affe777de8623697f62791fa5fa4"
  },
  "debian_image": "debian@sha256:a29215f6a35e51e22adffa17f89e9d2ef06214e64a2bad10d765c46aea49f11f"
}
```

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

本轮收口时没有待处理生产写操作或新增管理会话；正式服务、Poll/Delivery、在途及上游待收本轮均未查询，所以不声明当前为空或已登录。原d02d工作树保留在`31f46b2827db9353808ee11ff01ab7fdbf173c8e`且未修改；五个独立候选工作区保留源码及证据，原有其他工作树/未提交材料没有被reset、clean或删除。最终文档提交只固定这份交接，不能当作运行产品SHA。
