---
title: Agent 后端的三层架构：从循环到可维护系统
topic: ai-agents
source:
  - ../../raw/ai-agents/agent-backend-architecture.md
status: draft
updated_at: 2026-10-23
---

# Agent 后端的三层架构：从循环到可维护系统

## 一句话摘要

Hucci写代码 用一个 11 分钟的动手示例，把"Agent 就是循环"落成一个分层后端：领域层定义系统语言（session / tool / context），应用层编排流程，适配层翻译外部协议，组装根负责把具体实现接起来；并给出一个不常见的安全设计——工具调用前先用另一次模型调用审核。

## 核心判断

1. **Agent 本质是一个循环，难点不在循环本身。** 作者开篇把生产化问题收敛成三条：多模型适配、命令执行安全、Harness 可扩展性。定位：*00:00*。
2. **带工具调用的循环需要一次额外的审核调用。** 模型返回工具调用后，「另外调一次模型来审核工具调用是否合理」，审核通过才执行，不通过则把拒绝原因作为 tool result 交回模型。定位：*00:27*；拒绝原因回灌见 *08:22–08:25*。
3. **写成一个函数会烂掉。** 不分层的代价被具体化为两个场景：换模型协议要改多处；测流程必须真调模型，又慢又贵。定位：*01:08–01:25*。
4. **分层依据是"工作性质"，不是技术栈。** 领域层定义系统自己的语言，应用层负责编排，适配层把外部概念翻译成内部概念。三者的关系是单向的：「适配层和应用层是为领域层而生的」。定位：*01:34–02:40*。
5. **接口放应用层、实现放适配层，这是整个设计的骨架。** session store 声明在应用层，jsonl 实现在适配层；理由是可依赖接口而不依赖具体类，换实现和换测试替身都只动一处。作者称这是上一期讲过的依赖倒置。定位：*04:10–04:42*。
6. **context 用泛型输出类型，一个类型服务多种任务。** 回复用户时输出 assistant message，审核工具调用时输出 decision（能否执行 + 拒绝原因），差别只在 `output_type`。定位：*05:51–06:27*、*08:13–08:22*。
7. **厂商差异必须在适配层被吸收。** 同一概念在三家的名字分别是 `effort`（本系统）、`reasoning_effort`（OpenAI）、`output_config.effort`（Anthropic）、`thinking_level`（Gemini）。定位：*06:58–07:20*。
8. **两个角色可以配不同的模型。** assistant 与 tool call reviewer 各有 model setting，「reviewer 可以考虑用一个更便宜更快的模型」。定位：*07:37–08:52*。
9. **防死循环是应用层的显式参数。** `max_iterations` 与两个角色、两个 setting 一起挂在 agent service 上。定位：*08:35–08:55*。
10. **入口也是适配器。** 应用层不关心前端是命令行还是网页；HTTP 或 CLI 分别再写一个适配器。定位：*09:07–09:20*。
11. **组装根是系统里唯一知道具体类的地方。** 换模型、换存储、换入口都只改这一个文件（示例是 Claude 换 GPT）。定位：*09:24–09:56*。
12. **方法论可移植到任何系统，不止 Agent。** 先梳理流程，再把流程里的概念定义成领域语言，然后编排、定接口、实现适配、最后组装。定位：*10:14–10:28*。

## 八步工作流（00:27–00:57）

1. 新建对话或继续旧对话
2. 收到前端发来的用户消息
3. 把指令、可用工具、历史消息组织成上下文
4. 把上下文发给模型
5. 若回复含工具调用，另调一次模型审核是否合理（需要另一份上下文）
6. 审核通过则执行工具，结果连同上下文回传模型；不通过则返回拒绝结果
7. 重复 3–6，直到没有工具调用或达到循环上限
8. 把结果返回前端；过程中每条消息产生即持久化

## 领域模型

**session**（对话）：`id`（字符串，用于复用对话）、`messages`（列表）、`append()`。

**message 三种**：
- `user_message`：文本
- `assistant_message`：文本 + 可选的 `tool_call`
- `tool_result`：`tool_call_id`（对应哪次调用）+ `content`

**tool_call**：`id`、工具名称、调用参数。

**tool**：`name`、`description`、`parameters`，以及最重要的 `execute()`；输入是模型返回的参数，输出是 tool_result 的 content。适配层给出具体实现，示例是 bash tool。

**context**：`instruction`、可用工具、历史消息、`output_type`；作者说明还可扩记忆与 AGENTS.md，示例只留四种基础字段。

**model_setting**：模型 + 思考深度（本系统定义 6 档，与厂商实际档位不同）。

**两个角色**：
- `assistant`：有 instruction 与 tools，产出 `assistant_message` 类型的 context
- `tool_call_reviewer`：有 instruction，无工具，产出 `decision` 类型的 context

**应用层组件**：`session_store`（列出/存/取）、`model_client`（只有 `generate(context, setting)`）、`agent_service`（持有 model_client、session_store、两个角色、两套 setting、max_iterations；对外暴露新建对话、列出对话、发送消息）。

## 与已有条目的关系

这篇视频补的是**实现层**，两篇 harness 文章讲的是**概念与组件层**：

- [LangChain：harness 的原始定义与组件推导](agent-harness-langchain.md) 把 orchestration loop 称为 harness 的心跳；本视频给出这条心跳在代码里怎么分层、怎么注入依赖。
- [Agent Harness 解剖](agent-harness-anatomy.md) 列的 guardrails 与权限层，在本视频里被具体化为「审核调用 + decision 类型 + 拒绝原因回灌」这一条最小实现；两篇提到的 `max_iterations`、双模型、工具 schema 也都能在这里找到落点。
- *归纳*：如果要用一句话概括三者关系——两篇文章回答"harness 有哪些部件"，这个视频回答"这些部件怎么组织成不吃屎的代码"。

## 未解决问题

- ASR 有两处语义不明，未能可靠还原：*01:25*「计划前，又很慢」疑为"集成测试又很慢"；*05:24*「这些是构创开文使用了」疑为"这些是构建 tool 时用到了"。原文保留在 raw，未强行改写。
- 视频只给设计结论与目录结构，没有完整代码仓库或链接（*09:56* 提到"最后的目录差不多长这样"，截图中信息未转写）。
- `decision` 类型的具体字段只说了"能不能执行 + 拒绝原因"，未展开类型定义。
- 作者建议把设计结论交给 AI 实现并配合 TDD（*10:00–10:08*），但"接口限定好，代码质量就差不到哪去"是经验判断，没有给出验证方式。
- 视频属于「AI 工程」合集第 16 集，前 15 集未入库；*04:42* 提到的"上一期"具体是哪一集未核实。

## 来源

- 视频：https://www.bilibili.com/video/BV1winH6DEgt/（BV1winH6DEgt）
- UP 主：Hucci写代码（mid 1318868）
- 发布：2026-10-23 17:00，时长 11:13
- 抓取与转写：2026-10-23，faster-whisper large-v3-turbo（int8/CPU，238 秒）
- raw：`../../raw/ai-agents/agent-backend-architecture.md`（135 段带时间戳转写 + 同音误识对照表）
