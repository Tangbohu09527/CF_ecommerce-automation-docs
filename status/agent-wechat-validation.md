# agent-wechat V1 入口验证记录

- **验证时间：** 2026-07-30；合并转发消息补充验证记录于 2026-07-31；结构化 mention 结果于 2026-08-01 补充记录（均未记录具体时分）
- **验证对象：** `agent-wechat` V1 微信消息入口层
- **验证结论：** 微信消息入口层技术可行

## 验证范围

本次验证只覆盖微信消息进入 `agent-wechat` 时的登录、消息类型和入口标识能力。它不覆盖 CF Gateway、Hermes Agent、Skills、ERP/S6 接口、文件内容处理、正式归档或生产稳定性，也不代表企业 AI 自动化端到端链路已经完成。

## 已完成能力

- 微信登录。
- 私聊文本消息。
- 群聊文本消息。
- 文件消息。
- ZIP 文件。
- 引用消息。
- `sender` 识别。
- `chatId` 识别。
- 文件获取。
- 通过 API 读取消息。

## 结构化 mention 补充验证

验证时机器人当前微信显示名为 `Bot_测试版`。

| 测试 | 操作 | 原始结果 | 结论 |
| --- | --- | --- | --- |
| 1 | 真正通过微信成员列表选择 `@Bot_测试版` | `isMentioned=true` | 结构化 mention 成立 |
| 2 | 真正通过成员列表 `@` 群内其他成员 T | `isMentioned` 字段缺失 | 当前机器人未被 mention |
| 3 | 只复制或输入 `@Bot_测试版 手工文字对照`，未从成员列表选择机器人 | `isMentioned` 字段缺失 | 正文看似含 `@` 不构成结构化 mention |

最终标准化规则固定为：

```python
is_mentioned = raw.get("isMentioned") is True
```

字段缺失按 `false`。禁止：

- 根据正文中的 `@` 字符判断。
- 根据当前名称 `Bot_测试版` 判断。
- 根据机器人旧名称 `1024` 判断。
- 根据引用消息判断。
- 自动继承上一条 mention。

该结论是微信入口实测；Gateway 中的 `is_mentioned` / `is_self` 标准化代码已实现。它不代表 Adapter 已正式写入 Message Store，也不代表准入、Task、Hermes 或结果回传链路已经运行。

## 合并转发消息补充验证

### 当前支持

- 识别合并转发消息。
- 获取发送人。
- 获取外层标题。

### 当前限制

- 未支持展开内部聊天记录。
- 未支持自动提取内部文件。

合并转发属于增强解析能力。后续可增加 `forward parser`，但该增强尚未实现；它不影响普通微信文件经 `agent-wechat` 进入 Hermes 后续处理的现有架构方向。

## 未完成能力

- 实时事件机制：包括 WebSocket 接口可用性、事件范围、断线重连、去重和补偿机制，仍待研究和开发，不得表述为已完成。
- Adapter 到 Message Store 正式接线。
- Polling 与 Checkpoint。
- Admission Orchestrator；Message Store、Identity Mapping、Access Control、Employee Workspace 和 AI Thread 等已实现基础模块尚未被编排为正式准入链路。
- Context Builder 和 Task Queue。
- Hermes Agent 与 Worker Bridge 接入。
- Skills 系统及具体业务 Skill 的定义、实现和验收。
- 旺店通 ERP、旺店通 WMS 和 S6 接口验证与对接。
- 图片、Office、PDF、中文文件名、连续多附件、大小边界、失败重试和长期稳定性等未覆盖入口场景。
- 合并转发解析增强：展开内部聊天记录、自动提取内部文件和 `forward parser` 实现。
- 临时文件管理、ZIP 自动解压、OCR/视觉、Skill 处理、File Service 对接、产物回写和文件中心归档。
- 完整权限调用链、任务状态、审计闭环和端到端结果回传。

## 后续文件处理流程

以下是未来流程，不代表当前已经实现：

`微信文件 -> agent-wechat API -> Hermes -> 临时文件 -> 解压 -> OCR/视觉 -> Skill 处理 -> 文件中心`

实际实现必须遵守 [系统设计](../02_系统设计.md) 中 CF Gateway、Debian 权威控制面、File Service、权限和审计边界。第一阶段不建设独立 OCR，OCR/视觉能力是否引入仍需按实际需求评估。

## 当前状态

- **已完成：** 微信入口验证。
- **已完成：** 结构化 mention 三组对照验证。
- **代码已实现：** Gateway 微信适配基础，包括 HTTP Client、标准化、媒体 JSON / Base64 解码、文本消息真实发送字段和微信系统消息解析。
- **待开发：** Adapter 到 Message Store 正式接线、Polling / Checkpoint 和 Admission Orchestrator。
- **待开发：** Context Builder、Task Queue、Hermes 接入和端到端回传。
- **待开发：** 实时事件机制。
- **待开发：** 合并转发解析增强。
