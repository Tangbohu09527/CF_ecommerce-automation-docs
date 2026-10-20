# Validation Records

本目录保存带日期的执行记录，已完成的旧结果不被新记录覆盖。

## 当前记录

- [2026-10-20 本轮需求同步与只读代码核对](./2026-10-20-requirements-sync.md)：远端main/PR9—12、PR11/PR12语义吸收、固定源码与文档检查；无新增现场验收。
- [最新需求权威正文](../../requirements/confirmed-platform-scope.md)：2026-10-20 用户确认的当前范围、B 方案、分组分机、文件触发与日期查询；这是需求决定，不是本轮新增现场验收。原六阶段编号不变。
- [2026-10-20 总体状态证据](./2026-10-20-overall-status-evidence.md)：PR11 保留的总体需求、固定 SHA、PDF/PNG/正常 ZIP 出站及未完成边界；最后生产观察为 2026-10-19 北京时间 18:36，本轮未重验。
- [2026-10-20 统一部署与官方 Hermes 纯配置候选证据](./2026-10-20-configuration-only-candidate.md)：从 PR12 `3aa7bbbbe3ec3a1d756b06bdc5ed2d0265f6a041` 完整保留正文，仅新增顶部来源说明，补充 PR11，不整份覆盖。Gateway PR23 `57bb2c2830f292d5167f0bda449afafbe0a528fd` 的可信任务授权/返回链仍缺；PR24 `b3f2740289c6b1fa3208c352bc87b99b6f50cce4` 的隔离统一入口通过不代表三组件整机重启或生产部署完成。该记录内“并行 Skills/明早动作”等为当时安排，当前优先级以最新需求为准，不改写历史。
- [2026-10-11 总体状态证据](./2026-10-11-overall-status-evidence.md)：保留当时回退和未切换失败事实，不作为当前待执行步骤。
- [2026-09-17 Gateway 长任务回传](./2026-09-17-gateway-long-task-acceptance.md)：新 Gateway 限定现场验收，保留未完成边界。
- [2026-09-03 Enterprise Runtime Production Closeout](./2026-09-03-enterprise-runtime-production-closeout.md)：历史总体生产基线；旧 Gateway 镜像不是 9 月 17 日观察的新镜像。

## 命名

```text
<YYYY-MM-DD>-<release_id>-<scope>.md
```

重新执行应新增记录并说明与旧记录的 supersedes 或补充关系；不覆盖旧失败、不把新测试写成旧任务成功。

本轮文档同步未运行产品测试、安装、模型或现场验收，不新增虚构的生产 PASS。文档检查与 Git 同步状态单独记录；Draft PR 更新不代表默认分支已同步。

## 必填内容

- Purpose、scope、evidence date。
- 仓库 SHA、PR state、Image ID/registry digest 的准确类型、database revision 及证据来源。
- 实现、自动化测试、GitHub Actions、部署和现场验证分层。
- 前置条件、预期、实际、PASS/FAIL/BLOCKED/NOT_RUN。
- rollback、offline evidence、残余风险和责任边界；未独立核验的资料明确标记。
- 脱敏 correlation/evidence reference；未合并组件文档使用固定提交引用，不冒充 main。

## 安全

记录不得包含真实账号、联系人、群名、Chat/account/conversation ID、原始消息正文、QR、Token、Token hash/prefix、Cookie、Authorization、数据库 URL、密码、内网 IP、Windows 本地绝对路径或业务文件名。生产内部记录编号和合成测试编号只用于限定关联，不发布原始业务资料。

一次性生产运维路径只能进入对应带日期 Closeout；通用 Runbook 使用变量或路径模式。原始终端/截图由现场受保护存储保管，公开摘要不能虚构离线档案位置或校验和。
