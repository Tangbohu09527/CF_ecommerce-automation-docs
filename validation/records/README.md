# Validation Records

本目录保存带日期的执行记录，已完成的旧结果不被新记录覆盖。

## 当前记录

- [2026-09-17 Gateway 长任务回传](./2026-09-17-gateway-long-task-acceptance.md)：新 Gateway 限定现场验收，保留未完成边界。
- [2026-09-03 Enterprise Runtime Production Closeout](./2026-09-03-enterprise-runtime-production-closeout.md)：历史总体生产基线；旧 Gateway 镜像不是 9 月 17 日观察的新镜像。

## 命名

```text
<YYYY-MM-DD>-<release_id>-<scope>.md
```

重新执行应新增记录并说明与旧记录的 supersedes 或补充关系；不覆盖旧失败、不把新测试写成旧任务成功。

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
