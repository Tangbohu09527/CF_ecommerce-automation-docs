# 真实微信实时归档故障与修复索引（2026-10-22）

用户实测群消息没有自动出现，**现场验收未通过**。本记录纠正前一轮“只待用户标记”的状态，不改写10月21日历史观察或PR13已确认的Profile B需求、Skills后置决定。

第一断点：微信停在保存账号的Enter Weixin登录页，上游漏识别并沿用旧ChatOpen/登录状态，绿灯因此误报在线；原监视器反复杀微信。另有SQLCipher immutable忽略WAL及已真实复现的前端快照/增量游标竞态。591条未关联观察是首次基线，不解释为591条丢消息。详细证据只在所属组件和私有运维台账，不发布真实资料。

| 交付 | 固定事实与实际边界 |
| --- | --- |
| Gateway | [Draft PR28](https://github.com/Tangbohu09527/CF_agent-gateway/pull/28)，base PR27；运行代码`73eef5c899292afa1b317d9c89242893ff7139d8`，镜像`sha256:7dc2fc503b47cdd7c3b9295f8aa949b7cb4a04c87e1adac2f290df3d5c6a9499`；文档HEAD`853f53ef4a9c012e8fbac330138a9a759002200c`。修复快照完成前错误确认游标及白名单状态原因，未改schema、基线、权限、Profile或派发 |
| WeChat | [Draft PR14](https://github.com/Tangbohu09527/CF_agent-wechat/pull/14)，组件代码`180a11cfa32c5d58378c355776bc4687654acaba`，构建merge SHA`70d0546b34d038b7b5e6928659ad39c49b50a3bc`，文档HEAD`73e6e83ed3428ebe8fc5f11e0ff39c9123af00ee`；经用户单独批准只备份并替换API二进制、精确PID TERM。运行binary SHA256 `5330f4b2fe69f3deb6a2070aabf93b3a02a63f8ff15ec375c71777dd9c7db4ce`，微信PID/保存session保留，原容器镜像未换；recreate会丢失可写层热补丁 |
| 候选现场 | 17:31–17:32更新原独立console/proxy，备份与数据指纹通过。18:00仅修正旧环境文件覆盖的新版本字段；18:02:01复核代码/环境一致、候选messages=0、591观察/linked=0，22会话基线保留；18:02:27真实Edge证书、原登录、聊天/通讯录、新JS通过。准确内网地址和停止/回退入口只存既有本地台账，原HTTPS18445 `/console`书签不变 |
| 真实状态 | `logged_out / offline`，仅归档启用但`blocked`，不再用旧绿灯声称在线。补丁后至18:02有界日志无kill/spawn/unresponsive；这不证明所有未知界面都稳定。正式Poll退出、Dispatch运行、Delivery退出，状态/身份保持不变；候选只写独立库、正式档案只读 |

[Gateway固定故障记录](https://github.com/Tangbohu09527/CF_agent-gateway/blob/853f53ef4a9c012e8fbac330138a9a759002200c/docs/console-live/realtime-incident-2026-10-22.md)与[WeChat固定验证记录](https://github.com/Tangbohu09527/CF_agent-wechat/blob/73e6e83ed3428ebe8fc5f11e0ff39c9123af00ee/docs/validation/2026-10-22-live-observation-candidate.md)区分源码、部署、自动化和现场验收。加密一致性snapshot及SQLite backup完整性通过；回退演练未执行。

Console CI [39755470914](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39755470914)成功，含Linux实际浏览器/竞态/HTTPS与受限控制；微信原生构建[39755928451](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39755928451)成功。Gateway标准CI [39755470702](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39755470702)失败：2938 passed、7 skipped、既有5秒租约测试1 failed，后续迁移/PostgreSQL/V2显式步骤跳过，不算通过。未为全绿扩大本轮修改或削弱过期租约检查。

用户要求**扫码登录，不点Enter Weixin**，下班后先完成可独立工作。本轮未点击Enter或Switch Account、未走完整WS LoginPlan、未清runtime/checkpoint/基线、未换号或启动AI/发送。固定官方POST `/api/status/login`只读取屏幕上已有二维码，当前保存账号页没有二维码；安全导航到扫码页的受限操作尚未接通。下次先准备明确扫码路径并核对同一活动账号，再集中做私聊、连续群、其他会话摘要、浏览器断线补齐、历史阅读和样本级无AI验收。**A–F均未现场通过，2–5秒目标未测得**。

管理台、代理、原受限Controller保持运行；仅归档保持启用配置但登录受阻。原联系人、权限、分组、节点与Profile保留，候选权限不覆盖正式权限。全量现场盘点未完成、FileBrowser原密码未找回的既有状态保留。本次不扩展节点、Profile热更新、扫码核心、文件名、Skills、安装器或资源调度。
