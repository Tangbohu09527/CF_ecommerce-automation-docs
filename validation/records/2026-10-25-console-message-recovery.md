# 消息源页错位与实时未恢复索引（2026-10-25）

北京时间2026-10-25 **12:32**的组件现场结论：新Bot真实登录保持，消息源库读取受阻，**第一阶段实时验收未通过**。私聊、群聊样本未取得上游稳定事件，不能测出发生/接收、首次可读、提交、增量和当前浏览器显示的完整延迟；跨会话、断线补齐及历史阅读没有有效实际新增样本。10月23日自然群部分成功仅是当时观察，不能覆盖本次失败。

本记录只同步总文档索引，不连接或操作服务器。接续[活动账号隔离记录](./2026-10-23-console-account-isolation.md)，保留原日期，不改变[PR13确认需求](../../requirements/confirmed-platform-scope.md)、独立Profile B方案或Skills后置决定。

## 已确认断点与恢复边界

消息主库21个物理页与其实际逻辑页号不符，密文与旧WAL残页精确匹配且按其他页号通过HMAC；15个错位目的页符合跨代相同frame索引误配规律。旧deleted WAL/SHM与新路径WAL并存，已观察到跨代文件/索引不一致；具体形成时间和操作方仍未证明，不能直接归因于API。两个WAL没有当前有效commit，本轮没有拼接WAL、清基线或倒退checkpoint。

官方material元数据验证后，仅在私有隔离副本中恢复出含**598条历史消息**的加密subset，独立完整性检查通过。它不是当前完整commit，不含本次当天P1/G1样本，没有回写微信源库或导入Gateway；**subset完整性通过不等于实时恢复，不能用partial覆盖活动源文件。** 原始保全、覆盖缺口与共享索引副本变化限制见组件固定报告。

## 固定交付与实际运行

| 项目 | 固定引用与边界 |
| --- | --- |
| Gateway | [Draft PR31](https://github.com/Tangbohu09527/CF_agent-gateway/pull/31)，依赖PR30；候选实际应用源码`f84125169a6c0e2392c296f17226673b47ed6d3f`，镜像`sha256:0f64ef106ff320e0c50c423e6b3352485439ebea2d7762c99a0329db990e0392`。12:21至12:22只更新独立候选console/proxy，聊天区增加受阻/过期状态、独立时间和诊断入口；没有修复消息源库 |
| 微信入口 | [Draft PR16](https://github.com/Tangbohu09527/CF_agent-wechat/pull/16)，补丁源码`a2a6f1b371e42ccfd1d9e785e9647256a8d860cc`；实际API二进制SHA256仍为`8f56dd8f121f495a90e03043ecc022d26ba79a1232bb28b36cc9baae90e5d9e5`。本轮未替换API、重启微信或重建微信容器；可写层热补丁未固化为正式镜像，不含账号守卫或文件名修复 |
| 组件完整记录 | [Gateway固定恢复报告](https://github.com/Tangbohu09527/CF_agent-gateway/blob/01b3a991d745a28f1eecd2a95b08cf46822aeffa/docs/console-live/message-recovery-2026-10-25.md)，报告HEAD为`01b3a991d745a28f1eecd2a95b08cf46822aeffa`；该文档提交不替代实际运行应用源码`f84125169a6c0e2392c296f17226673b47ed6d3f` |

原候选HTTPS18445 `/console`书签、证书、管理员登录、配置和数据保留，准确地址及停止/回退方式仍在受保护现场台账，不在本仓库复制。候选和获准仅归档检查保持运行，但检查继续不等于消息采集成功；旧账号只保留历史、权限和Profile，不重新登录或测试。

当前页面定向检查12项中11项通过，指定私聊目标未在候选会话列表发现的一项未通过；不抹去该项。合成浏览器故障提示19项、live回归50项、快照/增量竞态7项、相关Python22项及离线解析器纯合成14项分别通过，不能替代真实验收。[标准CI40161293263](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/40161293263)及[Console CI40161293327](https://github.com/Tangbohu09527/CF_agent-gateway/actions/runs/40161293327)均已核对为成功，均对应上述`f841251…`。

## 后续与依赖

目前没有已验证的仅换API或保持活动连接直接替换源文件的修复路径。继续在私有副本核验完整恢复及范围；任何关闭客户端连接、重启微信或切换源库都须另提明确变更、风险和回退边界并获新授权，不在本索引执行。当前新Bot登录保留，不以在线绿灯代替消息链成功。

正式Poll、Dispatch、Delivery保持现场核对后的原状态；没有为本次排障开启新的AI准入、派发或发送。候选权限不代表正式授权，旧Bot权限不自动复制给新Bot。**第一阶段尚未通过；员工/群授权、独立Profile与Hermes文字回复/持续会话第二阶段仅登记，未开启。**

本总文档分支基于[Draft PR18](https://github.com/Tangbohu09527/CF_ecommerce-automation-docs/pull/18)的`00c15c9bcffb5fe5de16d42707663fe8ccd11043`，另依赖Gateway PR31；两份Markdown只追加结果与导航，不修改需求、业务代码、安装器或其他服务。文档检查不代表组件现场通过，Draft PR不表示main已同步。
