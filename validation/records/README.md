# Validation Records

本目录保存不可变的带日期执行记录。

## 当前记录

- [2026-09-03 Enterprise Runtime Production Closeout](./2026-09-03-enterprise-runtime-production-closeout.md)

## 命名

```text
<YYYY-MM-DD>-<release_id>-<scope>.md
```

重新执行必须新增记录，并用 `supersedes` 关联旧记录，不覆盖旧结果。

## 必填内容

- Purpose、scope、evidence date。
- repository SHA、PR state、image digest、database revision。
- repository implementation、automated tests、GitHub Actions、deployment、production validation 分层。
- 前置条件、预期、实际、PASS/FAIL/BLOCKED/NOT_RUN。
- rollback、offline evidence、残余风险和 owner。
- 脱敏 correlation/evidence reference。

## 安全

记录不得包含真实账号、联系人、群名、Chat/account/conversation ID、消息正文、QR、Token、Token hash/prefix、Cookie、Authorization、数据库 URL、密码、内网 IP、Windows 本地绝对路径或业务文件名。一次性生产路径只能进入对应带日期 Closeout；通用 Runbook 使用变量或路径模式。
