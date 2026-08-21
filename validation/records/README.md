# Validation Records

本目录保存从[Production Validation Checklist](../production-validation-checklist.md)生成的不可变执行记录。当前仓库尚未把旧叙述性证据改写为新格式；历史记录见[历史证据索引](../history/README.md)。

## 命名

```text
<YYYY-MM-DD>-<release_id>-<scope>.md
```

日期使用执行开始的 UTC 日期，`scope` 例如 `production-validation`、`cfserver-reboot` 或 `hermes-recovery`。同一测试重新执行应创建新记录，并用 `supersedes` 关联旧记录，不覆盖旧结果。

## 必填证据

- `record_id`、Checklist 版本、test case ID 和 PASS/FAIL/BLOCKED/NOT_RUN。
- 环境、UTC/本地起止时间、operator、reviewer 和 change ticket。
- 每个仓库的 commit/tag、镜像 digest、Compose config hash、migration/schema。
- Hermes 版本、安装包 hash 和 Profile 引用。
- 前置条件、脱敏测试数据、预期、实际和清理/回滚。
- 脱敏 Message/Dispatch/Response/Outbox correlation ID；来源 `localId` 只记录不可逆脱敏引用。
- 权威命令/runbook 链接、日志/只读查询/截图的受控证据引用。
- 前后指标、残余风险、签核与 `supersedes`。

## 安全

记录不得包含真实消息正文、员工/群 ID、二维码、Cookie、Token、数据库连接串、文件内容、真实文件 hash 或宿主绝对路径。敏感原始证据放在仓库外的受控存储，本文只保存可审计引用。
