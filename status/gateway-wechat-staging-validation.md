# CF_agent-gateway V1 Staging 微信文本闭环验证记录

- **状态日期：** 2026-08-04
- **验证对象：** `CF_agent-gateway` 微信文本消息、权限准入、Hermes 调度和结果回传链路
- **验证基线：** V1 Staging 当前代码
- **验证环境：** Debian 13 Staging + `agent-wechat` Docker + CF Agent Gateway Worker + Windows AI 主机 Hermes API

## 验证结论与边界

V1 Staging 已完成真实微信文本消息 AI 闭环：获准员工发送的文本消息能够进入 Gateway，经过身份和权限准入，绑定 AI Thread，调用 Windows AI 主机上的 Hermes API，并把 Hermes 文本响应发送回原微信会话。

该结论只表示当前 Staging 文本链路已闭环，不表示生产上线、完整企业业务自动化、附件处理或 Skill 执行已经完成。

**已完成 / 已验证：**

- 微信文本消息 Polling 与 Checkpoint。
- Message Store。
- Identity Resolution / Identity Mapping。
- Permission Admission / Access Control。
- Employee Workspace 与 AI Thread 基础管理。
- Hermes API Client、消息 Dispatch、Response Relay 和 Hermes Thread Binding。
- `agent-wechat` 文本回复回传。
- `is_self=true` 的自发消息防回环。

**本次未完成，不得写成已实现：**

- 图片理解、图片附件传递及其他附件的系统级传递。
- 文件消息端到端处理、Office / PDF 处理和正式归档。
- OCR、ZIP / 其他压缩包自动解析。
- 企业知识库。
- Skill 自动执行及具体业务 Skill。
- Context Builder、Task Queue 和 Worker Bridge 的目标架构完整链路。
- 生产环境自动部署、生产高可用和长期稳定性验收。

## 已验证真实链路

```text
员工微信
  -> agent-wechat
  -> CF_agent-gateway WeChat Polling
  -> Message Store
  -> Identity Resolution
  -> Permission Admission
  -> Employee Workspace
  -> AIThread
  -> Hermes Dispatch
  -> Hermes API
  -> Hermes Response Relay
  -> WeChat Outbound Sender
  -> 微信回复
```

目标架构中的 Context Builder、Task Queue 和完整 Worker Bridge 仍未实现。本次 V1 Staging 文本闭环是当前可运行的有限链路，不应据此把目标任务、文件或 Skill 架构标记为已完成。

## Hermes 集成验证

已验证 Gateway 能够：

1. 使用 Hermes API Client 调用 Windows AI 主机上的 Hermes API。
2. 将获准微信文本消息 Dispatch 给 Hermes。
3. 保存并使用 Gateway AI Thread 与 Hermes Runtime Thread 的绑定。
4. 接收 Hermes 文本响应并交给 Response Relay。
5. 按原 `chatId` 生成出站发送请求并把结果返回原微信会话。

`hermes_thread_id` 已在本次文本闭环中建立运行时绑定，但它仍是可重建的运行时标识，不替代 Gateway 权威的 `ai_thread_id`。

## WeChat Outbound 契约修复

`agent-wechat` 文本发送接口为：

```http
POST /api/messages/send
```

当前正确请求体：

```json
{
  "chatId": "chat_example",
  "text": "reply_example"
}
```

旧实现使用 `content` 字段，和真实接口契约不一致；现已修复为 `text`，并通过真实微信回复回传验证。示例值均为脱敏占位值。

## Self Message Echo Loop 防护

机器人回复进入微信后，也可能被后续 Polling 再次读取。当前 Polling 在标准化和 sink 之前，对 `RawWechatMessage.is_self` 或映射载荷中的 `isSelf` 执行严格 `True` 检查；命中后直接过滤：

- 不进入 sink / Message Store 入站处理。
- 不进入 Identity Resolution、Permission Admission 或 Employee Workspace / AI Thread 处理。
- 不调用 Hermes。
- 仍推进对应 Checkpoint，避免反复读取同一自发消息。

该顺序阻止“机器人回复 -> Polling -> 再次调用 Hermes -> 再次回复”的回环，同时保持当前 Polling 链路的 at-least-once delivery 语义。其他非 self 消息仍遵循“持久化成功后推进 Checkpoint”的规则。

## 身份与权限验证边界

- 未配置身份的消息可按 Message Store 和拒绝策略保留所需记录，但不得进入 Hermes 执行链。
- 已授权测试身份在当前 User Policy、Gateway Policy 和 `normal` 风险策略下可以进入 Employee Workspace、AI Thread 与 Hermes 文本调用链。
- 本记录只使用脱敏测试身份，不保存真实微信 ID、账号、Token、Cookie 或登录数据。
- Identity Management 管理界面、完整岗位 / 群 / Skill / 文件权限矩阵和高风险审批仍未完成。

## 已知实现偏差：微信群线程键

当前已确定的目标设计保持不变：微信群聊 AI Thread 必须按以下键隔离：

```text
bot_account_id + group_chat_id + sender_id
```

审计发现 Gateway V1 当前 `thread_keys` 行为忽略 `sender_id`，现有测试也表明同群不同员工可能复用同一 AI Thread。这和[系统设计](../02_系统设计.md#物理微信会话与-ai-线程)及[员工工作区与 AI 会话线程设计](../design/employee-workspace-design.md#5-ai-thread-映射规则)不一致。

该差异是已知实现偏差，不是设计变更，也不能作为群聊员工上下文隔离已经验收的依据。在 Gateway 修正并补充“同群不同员工不复用 AI Thread”的测试前，群聊多员工线程隔离保持待修复 / 待复验。若未来确需改为整群共享，必须先更新[技术决策记录](../05_技术决策记录.md)，说明安全、权限、上下文和迁移影响。

## 质量检查

当前 V1 Staging 代码基线已通过：

```text
pytest: 393 passed
ruff: passed
git diff --check: passed
```

这些检查证明当前代码基线和限定 Staging 文本闭环通过验证，不替代生产容量、故障恢复、长期运行、附件或业务验收。

## 最终结论

V1 Staging 微信文本消息 AI 闭环已经完成，包括 Hermes 调用、响应回传、运行时线程绑定和 self message 防回环。当前仍不得表述为图片或文件处理完成、Skill 可自动执行、企业知识库可用、完整业务自动化上线或生产环境已自动部署。
