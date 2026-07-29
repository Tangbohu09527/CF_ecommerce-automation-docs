# 消息与任务流程

> 本文描述目标流程，不代表所有组件已经实现。当前只有 `agent-wechat` 的部署、微信登录、联系人/聊天/消息读取和消息发送完成基础验证；CF Gateway、Hermes、Skills、企业系统接口和文件自动处理仍处于待设计、待接入或后续规划状态。

## 普通问答

### 目标流程

```mermaid
sequenceDiagram
    participant E as 员工
    participant W as agent-wechat
    participant G as CF Gateway
    participant H as Hermes

    E->>W: @AI 查询问题
    W->>G: 转发消息事件
    G->>H: 提交受控上下文
    H-->>G: 返回回答结果
    G-->>W: 返回待发送回复
    W-->>E: 在原微信会话回复
```

`agent-wechat` 的消息读取和发送能力已经验证，但从 Gateway 到 Hermes 再回到原会话的完整问答链路尚未完成。CF Gateway 还需承担来源校验、路由、安全隔离及与权威上下文和任务状态的衔接。

## 企业业务流程

以员工发送“查询雪域鲜稻库存”为例，目标流程如下：

```mermaid
sequenceDiagram
    participant E as 员工
    participant W as agent-wechat
    participant G as CF Gateway
    participant H as Hermes
    participant S as 库存 Skill
    participant B as 旺店通 ERP / S6 API

    E->>W: 查询雪域鲜稻库存
    W->>G: 转发消息事件
    G->>H: 提交请求、身份和权限约束
    H->>S: 调用获授权的库存 Skill
    S->>B: 按业务口径查询
    B-->>S: 返回业务数据或错误
    S-->>H: 返回结构化结果
    H-->>G: 生成回复结果
    G-->>W: 指定原会话和回复内容
    W-->>E: 微信群或私聊回复
```

### 数据口径边界

- 旺店通 ERP 是电商库存、订单、物流和寄样订单的当前职责系统。
- S6 负责财务、线下业务和对账；只有查询口径确实属于 S6 时才调用其接口。
- 旺店通 ERP、S6 的具体接口、字段、权限和数据时效均待验证。
- 库存 Skill 尚未实现。示例只说明目标调用关系，不表示已经可以查询真实库存。

### 执行约束

1. CF Gateway 和权威控制面先校验会话、发信人和允许的能力。
2. Hermes 只选择当前任务获授权的 Skill，不直接持有企业系统的任意操作权限。
3. Skill 通过明确接口查询或执行，并返回结构化结果和错误分类。
4. 查询结果先写入权威任务状态，再由 `agent-wechat` 返回原会话。
5. 写入、删除、改价、批量操作等高风险动作必须增加人工确认和幂等保护。

## 文件处理流程

### 后续规划流程

```mermaid
flowchart TB
    E["员工发送产品资料压缩包"] --> W["agent-wechat<br/>附件能力待技术验证"]
    W --> F["文件中心 / File Service<br/>受控接收、登记、权限与审计"]
    F --> V["OCR / 视觉解析<br/>按实际需求评估，第一阶段不建设独立 OCR"]
    V --> K["知识库<br/>后续规划，形态与边界待确认"]
    K --> S["文件处理 Skill<br/>待定义与实施"]
```

该流程是**后续规划能力，不代表当前完成**。现阶段只确定正式文件访问必须经过 File Service；FileBrowser Enterprise 是面向人员的文件管理入口，不能替代自动化访问的权限检查和审计。

在进入实现前仍需确认：

- `agent-wechat` 对 ZIP、中文文件名、连续多附件和失败重试的兼容性。
- 压缩包大小、类型、解包路径、安全检查和恶意内容处理规则。
- 原件、过程产物、解析结果和最终文件的保留及关联规则。
- 是否确有 OCR 或视觉模型需求；第一阶段不建设独立 OCR。
- 知识库的业务范围、数据权限、更新方式和检索边界。
- 文件处理 Skill 的输入、输出、幂等性、失败分类和人工确认点。

## 失败与回传原则

- 任一环节失败都应返回明确状态，不伪造成功结果。
- 消息、任务、文件、Skill 调用和结果回传应使用可关联的 `trace_id` / `task_id`。
- Windows AI 节点离线时，Debian 已接收的消息、文件和任务不得丢失。
- 微信发送失败时保留待发送结果并按后续确定的策略重试或转人工处理。
