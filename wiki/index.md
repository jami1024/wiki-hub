# Wiki Index

> 这是 Wiki 的入口。新增条目时按主题分组，并链接到对应的 `wiki/<topic>/<slug>.md`。

## 条目

### ai-agents

- [Agent Harness 解剖：模型之外的工程才是产品](ai-agents/agent-harness-anatomy.md) — Akshay Pachaar（2026-04-06）：解释模型外围系统如何通过工具反馈、上下文管理、状态恢复和验证支持多步任务，并讨论模型变强后如何简化控制；厂商机制与性能数字保留为作者转述。
- [Agent 后端的三层架构：从循环到可维护系统](ai-agents/agent-backend-architecture.md) — Hucci写代码（B 站，2026-10-23）：从更换模型和测试流程的困难出发，解释领域/应用/适配三层、接口与组装根如何隔离变化；工具调用前的模型审核是示例方案。

两篇均于 2026-10-27 按 `library` 0.2.0 重写，基于已有 raw 梳理论证主线、关键机制与适用边界。

## 主题说明

- `ai-agents`：agent 架构、harness、上下文工程、工具与记忆系统。
