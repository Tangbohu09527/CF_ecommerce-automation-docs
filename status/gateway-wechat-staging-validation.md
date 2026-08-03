# CF_agent-gateway Debian Staging 真实微信联调验证记录

- **验证日期：** 2026-08-03
- **验证对象：** `CF_agent-gateway` 微信轮询 Runtime 与权限执行链
- **已验证版本：** `587f59f feat: wire wechat polling runtime`
- **验证环境：** Debian Staging

## 验证范围与边界

本次验证确认 Gateway 微信入口与权限执行链已在 Debian Staging 通过真实微信消息联调。结论只覆盖以下链路，不代表生产上线，也不代表完整 AI Agent 闭环。

**已完成：**

- 微信消息接入。
- Message Store。
- Identity。
- Access Control。
- Admission。
- Employee Workspace。
- AI Thread。

**未完成：**

- Hermes Runtime 接入。
- AI 回复回传微信。
- Skill 执行链。

Context Builder、Task Queue 及其他未在本记录列为“已完成”的组件，仍以[当前开发进度](./current-progress.md)为准。

## 运行环境

| 项目 | 已验证环境 |
| --- | --- |
| 操作系统 | Debian 13 |
| Python | 3.13.5 |
| 微信入口 | `agent-wechat` Docker 容器 |
| Gateway | `CF_agent-gateway`，与 `agent-wechat` 同机运行 |

安装、配置和启动命令见[部署运维](../04_部署运维.md#cf_agent-gateway-debian-staging-部署验证2026-08-03)。

## 已验证真实链路

```text
微信
  -> agent-wechat
  -> Gateway Runtime
  -> Polling
  -> Checkpoint
  -> Message Store
  -> Identity Mapping
  -> Access Control
  -> Admission
  -> Employee Workspace
  -> AI Thread
```

该链路止于 Gateway 的 AI Thread，不包含 Hermes Runtime、Skill 执行或微信结果回传。

## 首次启动检查点验证

首次以 `bootstrap_mode=latest` 启动，结果为：

```text
messages_seen: 256
messages_skipped_by_checkpoint: 256
messages_processed: 0
```

该结果验证首次启动会建立当前消息高水位，不消费历史消息。

## 未配置身份的权限拒绝验证

- **测试消息：** `runtime测试2`
- **Message Store：** 保存成功。
- **Employee Workspace：** 未创建。
- **AI Thread：** 未创建。

该结果验证未知身份消息允许持久化，但禁止进入执行上下文。

## 已授权身份的准入验证

测试身份与权限：

| 项目 | 值 |
| --- | --- |
| 微信 ID | `wxid_z9few7e31a7p22` |
| Enterprise Identity | `EMP_TEST_001` |
| user policy | `enabled` |
| gateway policy | `enabled` |
| `allowed_risk_levels` | `normal` |

- **测试消息：** `runtime测试5`
- **创建记录：** `employee_workspaces`、`ai_threads`、`thread_source_bindings`。

该结果验证测试身份通过当前策略后可以建立 Employee Workspace、AI Thread 与来源绑定；它不表示 Hermes 或 Skill 已执行。

## Hermes 当前状态

AI Thread 已创建，但 Hermes Runtime 尚未接入，当前 `hermes_thread_id` 为空。下一阶段为 Gateway Hermes Adapter。

## 验证结论

`CF_agent-gateway` commit `587f59f` 已在 Debian Staging 串通真实微信消息进入 Gateway 后的持久化、身份映射、权限判断、准入和员工线程建立链路。当前不得将该结果表述为生产上线、完整 AI Agent 闭环、Hermes 已运行、AI 已回复微信或 Skill 已执行。
