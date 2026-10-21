# 微信管理台实时与受控运行增量（2026-10-21）

接续 PR15 的管理台交付索引，保留 PR13 已确认的 B 方案与 Skills 后置决定。本次需求增量为普通消息持续归档/自动显示、真实在线灯、已有 AI 节点展示、新主机向导与分项测试、受限微信服务控制和分层系统状态，不改变正式员工权限。

## 可用结果与准确版本

Gateway [Draft PR27](https://github.com/Tangbohu09527/CF_agent-gateway/pull/27)，分支 `codex/feat-console-live-operations`，base 为未合并 PR26 `codex/feat-wechat-management-console`。CFserver 独立候选已更新并保留运行，使用原 HTTPS 18445 `/console` 书签及原登录；不依赖 Windows 终端。内网地址、账号、私有配置、密钥和真实资料不进入本仓库。

- 实际运行源码：`b7d33d4446a3a3b07076c3fe65f0f678e12887e7`。
- 观察到的本地镜像 ID：`sha256:f71c1f1d964c2c1546abe183b16e104b4c01a306dfc99a05980a72d82e0e5d65`，不是 registry digest。
- Gateway 文档收尾 HEAD：`562cf676e90af2f34250ab321f08a7b83715bda7`，仅文档变化，不称新运行镜像；[固定交付记录](https://github.com/Tangbohu09527/CF_agent-gateway/blob/562cf676e90af2f34250ab321f08a7b83715bda7/docs/console-live/delivery-2026-10-21.md)包含使用、停止、回退和证据。

北京时间 **2026-10-21 17:43–17:52** 现场只读核对与候选操作：

| 项目 | 实际结果与保留边界 |
| --- | --- |
| 微信在线灯 | 新鲜认证/进程状态为已登录，来源账号一致，实际 Edge 绿灯；20 秒过期/失败变灰，不沿用早先离线缓存结论 |
| 采集 | 候选仅归档已运行；首次近期非隐藏会话窗口仅建基线，异常身份行显示 `partial`/缺口并跳过；不清正式 checkpoint、不重放旧记录、不下载全部原件 |
| 页面增量 | 持久稳定游标、断线补齐、同秒/迟到/重复、跨会话摘要、历史位置和退出隔离有合成实际浏览器验证；用户本次暂不发送标记，真实私聊/群显示延迟仍待现场验收 |
| 已有 AI 主机 | 原 Gateway 业务连接从 CFserver 通过网络、TLS、认证与错误密钥拒绝检查；未调用模型。管理连接未配置，不能称 Profile 创建可用 |
| 新主机向导 | 业务连接随机密钥、官方 API Server 五字段教程与只读测试已接后端；管理 REST/RPC 与业务入口分别显示，新电脑持久管理入口仍待配置/验证 |
| 受控微信操作 | 固定 Unix socket Controller，无 Web Docker socket/sudo/通用 Shell；已在线启动动作保留现状。当前目标缺登录前身份保护，扫码禁用；supervisor 应用启动未核验 |
| 分层状态 | 微信/采集、Gateway/候选库、正式 worker 与只读任务摘要、AI 业务/管理分别展示时间和来源；FileBrowser 容器运行健康，但候选无共享网络/获准健康连接，鉴权/空间/挂载/可写性未知 |
| 候选持久化 | 实际重建后原登录、节点/操作配置、采集开关、账号及旧会话基线保留，采集恢复成功检查；不等同整机重启或高可用 |

正式 Poll/Delivery 原本退出，Dispatch 原本运行，均保持原状态；指定正式容器的身份、镜像、启动时间和状态前后不变。只对候选应用必要迁移，没有正式 schema/权限迁移、AI 准入、历史重派、发送、微信退出或 Hermes 升级。采集不授予联系人权限，候选未关联不代表正式无权限。

## 安全登录候选与检查

WeChat 独立 [Draft PR13](https://github.com/Tangbohu09527/CF_agent-wechat/pull/13) 基于原名 PR9；代码 `84f2bbe077e7a48a6b4a5e9f74de82239fb185a1`，文档收尾 HEAD `5a5387c1b921782c077434f558d28b174d2e689c`。固定官方源码 `96c2e90c497d999bf5189a134a9b3b293731f42e` 叠加 `expected-account-v1`：缺身份/切号/账号不符拒绝接管，保留 session、keys、checkpoint，认证 API 声明能力。Python 3 与原生 Rust 8 项、官方 Cargo.lock 组合编译通过，[PR CI 39578604354](https://github.com/Tangbohu09527/CF_agent-wechat/actions/runs/39578604354) 成功。该候选未部署正式镜像、扫码或退出；错误扫码仍可能改变微信客户端自身状态，不保证自动切回。

实际运行 Gateway SHA 的 [普通完整 CI 39578393546](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39578393546)、[管理台浏览器/HTTPS/Linux 固定控制 39578393476](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39578393476)、[官方 Profile B 39578393518](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/39578393518) 均成功。Windows PowerShell 5.1 与 7 的启动、访问、登录、停止通过，BOM 保留。合成实际浏览器原流程/UX/节点/实时分别 48/18/24/31 项；截图全为合成数据，无真实正文或身份上传。正式现场模型执行、微信发送与扫码没有运行。

## 后续边界

真人方便时完成测试私聊/群标记与新 Hermes 配置，再测实际延迟/连接。正式微信核心安全 overlay 替换和离线扫码必须另行具体授权；当前在线账号保持。管理台、固定 Controller 和候选仅归档保留运行，停止/回退命令在组件交付记录，不能按旧规则全部关闭。

正式权限仍不迁移；旧会话 SOUL 即时刷新、FileBrowser 可信任务/附件 B、跨节点迁移/删除、完整群/成员、历史补采及完整新机管理入口继续分别未完成。本次不继续 Skills、资料加工、群发、自动调度或整套安装器。
