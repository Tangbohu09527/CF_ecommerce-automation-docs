# Deployment 文档索引

> 文档状态：当前生产部署入口
> 最近复核：2026-08-21

| 文档 | 用途 |
| --- | --- |
| [Deployment Guide](./deployment-guide.md) | 新机器、Docker、Hermes、微信登录、配置、升级与回滚 |
| [Timezone Policy](./timezone-policy.md) | Host、Container、Database 与 Application 的时区基线 |
| [生产拓扑](../docs/deployment/production-topology.md) | 当前部署事实与未来拓扑摘要 |
| [部署运维总入口](../04_部署运维.md) | 跨仓库发布、运维和证据入口 |

`docs/deployment/` 下标明 2026-08-11 的文件是 Staging 历史快照，不得直接作为当前生产命令。组件级 Compose、migration、登录脚本和健康接口必须来自目标版本的发布清单及对应组件仓库。
