# Deployment 文档索引

> 文档状态：当前生产部署入口
> 导航同步：2026-10-20；本轮未部署或实时核验。

| 文档 | 用途 |
| --- | --- |
| [Deployment Guide](./deployment-guide.md) | 新机器、Docker、Hermes、微信登录、配置、升级与回滚 |
| [Timezone Policy](./timezone-policy.md) | Host、Container、Database 与 Application 的时区基线 |
| [生产拓扑](../docs/deployment/production-topology.md) | 当前部署事实与未来拓扑摘要 |
| [部署运维总入口](../04_部署运维.md) | 跨仓库发布、运维和证据入口 |

最新[用户确认](../requirements/confirmed-platform-scope.md)将简部署/恢复入口后置，最终要求保留；PR12[最终统一入口/官方Hermes证据](../validation/records/2026-10-20-configuration-only-candidate.md)不等于真实新机/整机重启/可信任务返回完成。本轮只同步文档。

`docs/deployment/` 下标明 2026-08-11 的文件是 Staging 历史快照，不得直接作为当前生产命令。组件级 Compose、migration、登录脚本和健康接口必须来自目标版本的发布清单及对应组件仓库。[9月3日Closeout](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)为历史生产基线；最新状态按[状态矩阵](../status/current-status.md)，不推断当前在线。
