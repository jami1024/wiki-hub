# Wiki Index

> 这是 Wiki 的入口。新增条目时按主题分组，并链接到对应的 `wiki/<topic>/<slug>.md`。

## 条目

### ai-agents

- [Agent Harness 解剖：模型之外的工程才是产品](ai-agents/agent-harness-anatomy.md) — Akshay Pachaar（2026-04-06）：解释模型外围系统如何通过工具反馈、上下文管理、状态恢复和验证支持多步任务，并讨论模型变强后如何简化控制；厂商机制与性能数字保留为作者转述。
- [Agent 后端的三层架构：从循环到可维护系统](ai-agents/agent-backend-architecture.md) — Hucci写代码（B 站，2026-10-23）：从更换模型和测试流程的困难出发，解释领域/应用/适配三层、接口与组装根如何隔离变化；工具调用前的模型审核是示例方案。
- [Code Mode 一年后的复盘：为什么胜出的是"让模型写代码"](ai-agents/next-token-ep07-after-code-mode.md) — The Next Token 第 7 期（Dillon、Rhys、Sunil 与 Matt Carey，节目自述 2026-10-12）：从 MCP 工具挤爆上下文讲到 Code Mode 的定义、源流与 Cloudflare 的 search+execute 实现，记录执行环境选型、权限与异步化分歧，以及对"之后是什么"（状态、执行位置、Cap'n Web、resource-only MCP）的不同预测；后半段讨论个人 agent 与 agent 互刷的 slop highway。raw 为带时间戳字幕稿，正文本 `hh:mm:ss` 可回查原视频；Cloudflare 博客、OpenCode 2.0、Pi v0.99/v1.0 发布与 Earendil 转向已外部核实，Claude Code code mode 状态仍存疑。

前两篇于 2026-10-27 按 `library` 0.2.0 重写，基于已有 raw 梳理论证主线、关键机制与适用边界；第三篇于 2026-10-31 新增，raw 为 YouTube 自动字幕清洗稿（未人工校对），文中断言与预测已逐条标注。

## 主题说明

- `ai-agents`：agent 架构、harness、上下文工程、工具与记忆系统。
