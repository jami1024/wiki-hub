# Wiki Index

> 这是 Wiki 的入口。新增条目时按主题分组，并链接到对应的 `wiki/<topic>/<slug>.md`。

## 条目

### ai-agents

- [LangChain：harness 的原始定义与组件推导](ai-agents/agent-harness-langchain.md) — Vivek Trivedy（LangChain，2026-03-10）：从模型能力缺口反推六组 primitive（filesystem / bash / sandbox / memory / context rot / 长时程），并含与 Akshay 篇的逐维对照与 Ralph Loop 归因冲突。**建议先读这条**。
- [Agent Harness 解剖：模型之外的工程才是产品](ai-agents/agent-harness-anatomy.md) — Akshay Pachaar（2026-04-06）：跨厂商汇总 11 项组件（原文称 12 项）、单轮七步、七个架构取舍、脚手架隐喻与共演化。概念上承接上一条。
- [Agent 后端的三层架构：从循环到可维护系统](ai-agents/agent-backend-architecture.md) — Hucci写代码（B 站，2026-10-23）：用 11 分钟把 agent 循环落成三层后端（领域/应用/适配 + 组装根），含“工具调用前先用另一次模型审核”的安全设计；实现层，与上两条互补。

## 主题说明

- `ai-agents`：agent 架构、harness、上下文工程、工具与记忆系统。
