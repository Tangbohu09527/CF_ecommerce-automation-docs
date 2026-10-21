# 微信管理台候选交付索引（2026-10-21）

本轮在 Gateway PR25 基础上接续实现日常微信管理台：先从联系人、已发现群及聊天识别真实对象，再关联员工、配置两级权限和分组。保留 PR13 的 B 方案、组模板、每员工/群独立 Profile、Skills 后置决定；不另建用户中心或桌面客户端。

## 可用结果

Gateway [Draft PR26](https://github.com/Tangbohu09527/CF_agent-gateway/pull/26)，分支 `codex/feat-wechat-management-console`，base 为未合并 PR25 `codex/feat-profile-b-management`。实际产品运行源码 `caabee739f92ea2c927d318be88264a00de90a20`；后续 `48497865d7afb7cb2d4617145135514518df1447` 仅修复 HTTPS CI 生命周期，详细交付证据和使用说明由组件仓库维护。

CFserver 独立候选已经部署并保留运行，使用独立 Compose project、端口与 SQLite；局域网同源 HTTPS 管理入口脱离 Windows 终端。准确内网地址和登录方式仅在本轮私有交付消息提供，不在本仓库保存账号、IP、连接字符串或凭据。使用现有 CF CA，未关闭 TLS 验证；按用户许可仅对当前 Windows 用户添加既有公开 CA 信任。

| 交付项 | 实测范围 |
| --- | --- |
| 通讯录 | 独立原生 contacts 接口扫描 200/195/53/0 四页，共 448 条来源资料，普通联系人视图 430 条；短页继续；来源账号 + username 绑定，非 alias/昵称合并 |
| 日常管理 | 备注优先、昵称、alias、真实头像或占位、稳定标识、来源与同步状态；从好友关联员工、两级权限、启停、分组；不要求手填来源 ID |
| 群 | 11 个已发现群，从详情登记及分组，关联模板/电脑/Profile；完整群来源与群成员接口未取得，不称“全部群”，不推断总成员数 |
| 聊天 | 22 个近期/档案会话；真实只读档案查询及页面阅读通过，抽查会话有 2220 条记录，支持后端分页、北京时间日期、具名发言者、引用与附件状态；不代表完整账号历史 |
| 候选权限 | 仅候选数据库生效；没有自动给真实好友开权限或创建 Profile。真实页面只读核对及打开配置表单；完整授权/分组写入在合成实际页面和后端验证 |
| 持久化与访问 | 实际 LAN Edge 28 项通过，首屏 20/20 头像加载，1440/1920/430 布局、刷新及实际候选整套重建后登录/资料/配置保留均通过 |

当前微信进程 `app_not_running`，同步使用上游仍可读取的已存资料库，页面明确“来源离线 · 已读取缓存”。未恢复微信，未把缓存状态称为在线。

## 测试和组件证据

Gateway 固定文档提交 `00a9d6d0522db1c20ffcc73f26ce562e0a4e9b83` 的[详细交付记录](https://github.com/Tangbohu09527/CF_agent-gateway/blob/00a9d6d0522db1c20ffcc73f26ce562e0a4e9b83/docs/wechat-console/delivery-2026-10-21.md)、[使用与契约说明](https://github.com/Tangbohu09527/CF_agent-gateway/blob/00a9d6d0522db1c20ffcc73f26ce562e0a4e9b83/docs/wechat-management-console.md)及[合成截图索引](https://github.com/Tangbohu09527/CF_agent-gateway/blob/00a9d6d0522db1c20ffcc73f26ce562e0a4e9b83/docs/wechat-console/README.md)由组件仓库维护。正式姓名、账号、标识、消息正文、头像内容、登录凭据和真实截图未上传 GitHub/CI，也未回传模型。

本地后端/迁移联合 80 项、字段/离线/具名筛选定向 27 项；合成真实浏览器完整 48 项、UX 18 项；Windows PowerShell 5.1 与开发 PowerShell 的启动、登录、访问、停止通过。head `48497865…` 的 [工作台 CI](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39540099645) browser 与真实 Docker/HTTPS 两项均 success，包括 16 项鉴权/读取/持久化检查与退出清理。其 artifact 固定的是 PR 合并测试对象 `25e45b0ddfd8be4b0ef64296035d28da65fc8dd3`，不混同部署 SHA。

本轮 console/HTTPS 检查未运行 Hermes 或业务模型；同提交独立官方 Profile B 浏览器 CI 是另一项证据，不能当真实微信生产派发通过。未修改 WeChat、FileBrowser 源码，也不重做原名修复或现场收件验收。

## 保留边界

候选通过现有微信只读接口与强制只读正式档案连接读取资料，正式 schema 未迁移；正式 Gateway、Poll/Dispatch/Delivery、微信登录账号、正式 Hermes 和 FileBrowser 均未变更，正式容器身份/镜像/启动时间前后比较未变。只读 GET 的上游内部状态观察及懒缓存行为在组件契约中说明。

没有发送消息、清未读、补采、重新派发历史任务或正式权限切换。旧会话 SOUL 即时刷新、FileBrowser 可信任务及附件 B、完整群与成员、self/窗口/停采历史缺口继续保留；本轮没有连接正式 Hermes 来演示创建 Profile。缺少可用隔离节点时显示未验证/不可用，不返回假成功。

本机演示和协议替身已停止，CFserver 获准候选保留运行；准确停止命令在组件交付记录。后续恢复微信、补采、正式管理生效或生产切换均需独立授权，本轮完成后不继续业务 Skills 或其他批次。
