# 2026-10-20 独立 Profile B 与管理 Web 隔离交付

## 结果与固定代码

需求继续采用 PR13 固定提交 `bde24c32e77e6af9175a3fbe0721c98c93b67b05`，未改变 D044—D053 或后续业务 Skills 范围。本记录补充后续获准的 A→B 实施结果，不把 PR13 当时的“仅文档同步”边界套用于本轮。

Gateway 候选 [Draft PR25](https://github.com/Tangbohu09527/CF_agent-gateway/pull/25)，分支 `codex/feat-profile-b-management`，固定提交 `510d904a2022275b13030fa2777130356d80dea9`；base 为 PR21 `codex/feat-enterprise-zip-return` @ `91a235fca08362d8c1440e808df56946be3cc1e8`，未合并 PR23/24。组件完整说明与启动入口见[README](https://github.com/Tangbohu09527/CF_agent-gateway/blob/510d904a2022275b13030fa2777130356d80dea9/docs/profile-b/README.md)，详细证据见[validation](https://github.com/Tangbohu09527/CF_agent-gateway/blob/510d904a2022275b13030fa2777130356d80dea9/docs/profile-b/validation.md)。各 PR 均未合并，未部署生产。

| 项目 | 已证明的结果 | 保留边界 |
| --- | --- | --- |
| A 创建 | 页面经真实 Gateway 管理后端创建甲乙、群 A/B 四个真实官方 Profile；重复请求幂等，未知同名拒绝覆盖，真实网络中断后安全接续 | 同机独立官方节点实测；异机 HTTPS 管理部署未验收 |
| A 路由与持续会话 | 官方回执、各 Profile 的 state.db 与实际消息历史独立核对；私聊续接，同群换获准发起人沿用群会话并重新验权 | Profile 不是 OS/文件沙箱；真实微信协议未切换 |
| A 模板 | SOUL/display.language 白名单真实同步，目标/已应用版本分开；失败不写成功；保留历史、合成记忆和非受管配置 | 旧普通会话保留原 SOUL，重启仍如此；新官方对照会话采用新版本，旧会话无损即时刷新未接通 |
| B 管理 | 中文同源 Web 与真实员工两级权限、员工/群分组、模板、节点、派生、持续会话和审计 API 已接通；未分组拒绝，不暗配 | 仅显式 opt-in、wechat-test 合成输入；未迁移生产身份、会话或文件权限 |
| B 页面验收 | 真实浏览器完成添加、授权、分组、创建、消息、同步；刷新和后端/节点重启后保留；断网提示/恢复、权限拒绝、窄屏及正常控制台检查通过 | 无删除/跨节点迁移/拷贝记忆假成功按钮；在途不强行终止或自动重放 |

官方 Hermes 固定未修改 `427d605d4985c2d49a795493ef6c5d66413057d1`，官方安装器完整锁定 `[all]` 环境，Windows 按平台展开依赖 59 条、Linux CI 53 条，均缺失 0、不匹配 0，源码干净。使用官方认证 REST 与 WebSocket JSON-RPC；节点运行官方 `hermes serve`，没有项目插件、EXE、通用远程 Shell 或额外常驻节点客户端。确定性模型及微信输入为明确替身，Gateway、数据库、Hermes 和 Profile 为真实组件。

## 验证口径

本地定向后端/迁移 102 项及最终管理契约 13 项通过；实际 Playwright 页面、官方磁盘/SQLite、官方 read_file、网络故障后的创建/同步恢复和未知同名拒绝均执行。交付脚本在固定提交再次实际启停并通过重启浏览器检查。

Windows 全套开发期回归为 2665 通过、198 跳过、18 失败，17 项涉及既有 POSIX 存储/链接语义，另 1 项为既有租约日志时序断言；不称全套通过。Linux 准确提交的 [Profile B 联合 CI](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39412486537) 已成功，实际 browser/native 制品 passed=true 且 children_stopped=true；[正常 CI](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39412486536) 也已成功：完整套件 2874 通过/7 跳过，专项迁移 47 通过、带既有数据的 PostgreSQL 迁移 1 通过、V2 端到端 6 通过，PostgreSQL 确认单一 head `20261020_01`。PR25 当前 27 项检查均成功，含质量和隔离真实容器端到端；不推导为生产部署。Linux 初次暴露的虚拟环境解释器软链接解引用问题已修复并由本次真实组件 CI 复验，未以补装依赖子集绕过。

官方本地 read_file 仅检查合成小文件及调用归属。PR23 中 FileBrowser 每调用可信 task 关联、共享 MCP `_meta`/Artifact/Response/Delivery/微信返回缺口保留；独立 Profile 不扩大文件权限。附件接入、真实微信、异机 HTTPS、旧会话即时刷新和破坏性迁移未验收或未接通。

## 本机复用与 WeChat 候选

本机 PowerShell 运行 `D:\Tangbohu09527\.cf-worktrees\profile-b-management\CF_agent-gateway\scripts\profile-b\start.ps1`，页面 `http://127.0.0.1:18760/manage`。监督器生成的本地 `runtime\demo\demo-secrets.json` 的 admin 值用于测试登录，不入 Git/前端。测试数据与隔离依赖已保留，无需重新安装；停止使用同目录 stop.ps1。自定义运行目录须在启动、浏览器测试和停止时使用同一 Runtime，详见组件 README。所有本轮本机测试进程已停止。

WeChat 复用 PR9 `7bd42b623d405956080a13481bbf4de21aa3477a`，没有运行时代码新修改或无必要重建；准备分支 `codex/docs-wechat-filename-prep-20261020`，固定提交 `6ba59c68ec5324caa54e15609bc52bfd8054e3e8`，[Draft PR12](https://github.com/Tangbohu09527/CF_agent-wechat/pull/12) 的 18 项 CI 成功。[现场准备报告](https://github.com/Tangbohu09527/CF_agent-wechat/blob/6ba59c68ec5324caa54e15609bc52bfd8054e3e8/docs/validation/2026-10-20-return-filename-preparation.md)记录兼容组合、准确文件/产物身份、回退和中文/空格/括号/同名不同内容样本。构建包有效至 2026-11-01 北京时间，真实私聊/群聊收件名、内容和目标仍待用户现场扫码、发送和核收，本轮未执行。

没有连接 CFserver、正式 FileBrowser/Hermes，未读取生产密钥，未操作正式 Hermes 安装/HOME/模型/进程或真实微信/Poll/Delivery，没有部署、合并或发布。FileBrowser 无新修改，总需求正文保持原确认版本。完成后不继续其他批次、业务 Skills、资料库或安装器。
