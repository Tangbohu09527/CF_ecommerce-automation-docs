# Operations 文档索引

> 文档状态：当前生产运维入口
> 导航同步：2026-10-20；本轮未执行恢复或生产核验。

| 文档 | 用途 |
| --- | --- |
| [Recovery Runbook](./recovery-runbook.md) | Docker、微信登录、Hermes 和消息无回复的故障恢复 |
| [Deployment Guide](../deployment/deployment-guide.md) | 部署、配置、升级和回滚 |
| [Timezone Policy](../deployment/timezone-policy.md) | 主机、容器、数据库和应用时区 |
| [部署运维总入口](../04_部署运维.md) | 跨仓库权威入口和当前恢复边界 |

最新[恢复需求](../requirements/confirmed-platform-scope.md)保留在线微信登录、离线再扫码，与旧forced-QR契约冲突待后续受控实现验证；统一恢复入口后置，不默认清runtime/重装升级或重做不明任务。

`docs/operations/` 下的文件是标明日期的 Staging 历史快照。生产操作必须使用目标版本发布清单中的 Compose、服务名、健康地址和管理命令。reboot、forced-QR、Gateway-only cutover 和回滚的[9月3日证据](../validation/records/2026-09-03-enterprise-runtime-production-closeout.md)保持原日期；当前状态与缺口见[状态矩阵](../status/current-status.md)，不作为本轮操作授权。
