---
title: LangChain：harness 的原始定义与组件推导（含与 Akshay 篇对照）
topic: ai-agents
source:
  - ../../raw/ai-agents/agent-harness-anatomy-langchain.md
  - ../../raw/ai-agents/agent-harness-anatomy.md
status: draft
updated_at: 2026-10-23
---

# LangChain：harness 的原始定义与组件推导

## 一句话摘要

Vivek Trivedy（LangChain，2026-03-10）用"从模型能力缺口反推"的方法定义 harness，并逐项推导出 filesystem、bash、sandbox、memory/search、context rot 对策、长时程执行六组 primitive；这份较早的原文是 Akshay Pachaar 一个月后那篇汇总文的主要概念来源。

## 源流关系（先读这条）

- LangChain 篇 2026-03-10，Akshay 篇 2026-04-06，**晚 27 天**。
- 两篇共享同一句口号：「If you're not the model, you're the harness.」Akshay 明确标注其出处为「LangChain's Vivek Trivedy」——归属正确，无剽窃争议。
- *归纳*：Akshay 篇的增量价值在**跨厂商横向汇总**（Anthropic/OpenAI/LangChain/CrewAI/AutoGen 的实现对比 + 11 项组件 + 7 个架构决策）；LangChain 篇的增量价值在**纵向推导**（为什么需要这个 primitive，从模型缺什么出发）。

## 核心判断（LangChain 篇）

1. **定义取"排除法"，边界清晰到可执行。**「A harness is every piece of code, configuration, and execution logic that isn't the model itself.」并列出五类具体构成：system prompts、tools/skills/MCPs 及其描述、打包基础设施（filesystem/sandbox/browser）、编排逻辑（subagent、handoff、model routing）、hooks/middleware（compaction、continuation、lint）。定位：*Can Someone Please Define a "Harness"?*
2. **harness 的存在理由是模型开箱做不到的四件事**：跨交互保持持久状态、执行代码、访问实时知识、搭建环境装包。这四点是全篇的推导起点。定位：*Why Do We Need Harnesses*。
3. **filesystem 是最基础的 primitive，不是可有可无的附属。**「The filesystem is arguably the most foundational harness primitive」——因为它同时解决三件事：工作区、上下文卸载、**多 agent 与人的共享协作面**（Agent Teams 依赖它）。定位：*Filesystems for Durable Storage*。
4. **bash/代码执行取代"为每个动作预建工具"。**「Bash + code exec is a big step towards **giving models a computer**」——模型可在运行时自己造工具，而非被固定工具集限制。定位：*Bash + Code as a General Purpose Tool*。
5. **sandbox 既是安全边界也是规模单位。** 按需创建、扇出到多任务、用完拆除；配套要求是 allow-list 命令 + 网络隔离 + 预装 runtime/git/test CLI/浏览器。定位：*Sandboxes and Tools*。
6. **"加知识"在权重不可改时只能靠上下文注入。** 记忆靠 AGENTS.md 类标准的文件（启动时注入，被 agent 编辑后重新加载），时效性靠 Web Search 与 MCP（如 Context7）突破 knowledge cutoff。定位：*Memory & Search*。
7. **harness 在当下主要是上下文工程的投递机制。**「Harnesses today are largely delivery mechanisms for good context engineering.」三种对策：compaction（接近上限时卸载+摘要）、tool call offloading（保留头尾 token、全文卸载到文件）、skills（**progressive disclosure**，解决启动即加载过多工具/MCP 导致性能下降）。定位：*Battling Context Rot*。
8. **长时程执行靠 primitive 复利，而非单一技巧。** filesystem+git 做跨会话账本；Ralph Loop 用 hook 拦截模型退出、在干净上下文里重注原始 prompt 逼其继续；planning + self-verification（hooks 跑测试套件、失败回喂错误）提供反馈信号。定位：*Long Horizon Autonomous Execution*。
9. **共演化会造成过拟合，但不该决定你的选型。**「this doesn't mean that the best harness for your task is the one a model was post-trained with」——证据：Opus 4.6 在 Claude Code 里的得分**远低于** Opus 4.6 配其他 harness；LangChain 自己只改 harness 就从 Top 30 到 Top 5。定位：*The Coupling of Model Training and Harness Design*。
10. **harness 的长期趋势：重要性下降，工程价值不下降。** 类比 prompt engineering——被吸收进模型的部分越来越多，但围绕模型智能设计系统的能力仍是刚需：「make any model more efficient regardless of its base intelligence」。定位：*Where Harness Engineering is Going*。

## 与 Akshay 篇的对照（核心差异）

| 维度 | LangChain / Vivek Trivedy（2026-03-10） | Akshay Pachaar（2026-04-06） |
| --- | --- | --- |
| 组织方法 | 从模型缺口**反推**（behavior → design），公理式 | 跨厂商**横向汇总**，清单式 |
| 组件划分 | 不设固定数量，六组 primitive | 宣称 12 项、实列 11 项（含 orchestration loop、output parsing、prompt construction、guardrails、error handling、subagent） |
| filesystem 地位 | **最基础 primitive**，独立成节 | 未单列；状态管理讲 typed dict / git checkpoint，progress 文件附带提及 |
| sandbox | 独立成节：安全 + 按需扩缩 + allow-list/网络隔离 | 仅一句"在沙箱环境执行" |
| bash 作为通用工具 | 独立成节，"给模型一台计算机" | 未成节（Tools 一节讲 schema 注册与校验） |
| skills | 作为 context rot 对策，progressive disclosure | **完全未提** |
| hooks/middleware | 作为独立的确定性执行层（compaction、continuation、lint） | 无独立层，散布在各组件描述中 |
| 记忆的实现锚点 | AGENTS.md 文件标准 + Web Search/MCP | CLAUDE.md/MEMORY.md、LangGraph Store、OpenAI Sessions、三级记忆层级 |
| ReAct 的称呼 | ReAct loop | TAO（Thought-Action-Observation）循环 |
| 共演化证据 | 一手可核验：Codex-5.3 prompting guide 的 apply_patch 案例、TerminalBench leaderboard | 一句断言 "Changing tool implementations can degrade performance"，无出处 |
| 反直觉结论 | 不必沿用模型 post-train 时的 harness | 未强调；主线是"脚手架会被拆除" |
| 生产策略细节 | 少（偏概念推导） | 多（Stripe 重试上限 2 次、~40 个工具门控、1k–2k token 子 agent 摘要等） |

*归纳*：两份清单不冲突，是**不同切面**。LangChain 切"模型缺什么 → 补什么 primitive"（存储/执行/环境/记忆/上下文/长时程），Akshay 切"一条请求在 harness 里经过哪些环节"（拼装→推理→解析→执行→打包→更新→循环）。前者解释**为什么有这些部件**，后者解释**这些部件如何串起来跑一轮**。

## 事实冲突（需回溯一手来源）

1. **Ralph Loop 的归属。**
   - Akshay：「Anthropic developed a two-phase "Ralph Loop" pattern」，指 Initializer Agent + Coding Agent 两阶段，链接 `anthropic.com/research/long-running-Claude`。
   - LangChain：链接 `ghuntley.com/loop/`，即 Geoffrey Huntley。
   - 第三方证据（已核实）：`ghuntley.com/loop/` 自称引用「the original ralph post」，把 Ralph 定义为"单仓库、单进程、每轮一个任务"的 orchestrator 模式，反对多 agent 微服务化；该页发布于 2026-01-17，早于 Anthropic 的相关材料。
   - **判断**：Ralph 概念源自 Huntley 的可能性更大；Anthropic 更可能是把该模式做成两阶段实现。Akshay 的 "developed" 表述过强，引用时应改为"Anthropic 采用的 Ralph 变体"。
2. **TerminalBench 排名**：两篇一致（只改 harness，Top 30 外 → 第 5），无冲突。但 LangChain 补了一个 Akshay 没有的关键事实：**Opus 4.6 在 Claude Code 里的分数远低于同一模型配其他 harness**——这直接反驳"官方 harness 最优"的直觉。

## 未解决问题

- LangChain 篇称 "Opus 4.6 in Claude Code scores far below Opus 4.6 in other harnesses"，未给具体分数与对比对象。
- 「previous blog」指向一条 X 链接（`x.com/Vtrivedy10/status/2023805578561060992`），是原始实验记录，未入库、未核实。
- deepagents 库的具体实现是否真按本文六组 primitive 组织，未核实。
- 两篇都未解决"skills 与 tools 的边界"——LangChain 把 skills 当上下文对策，Akshay 完全不提 skills；这个分歧在 2026 下半年是否有共识，当前资料无法判断。
- 本文的 `published_at` 取自页面作者卡片 "March 10, 2026"，JSON-LD 无发布日期字段，属单点证据。

## 来源

- raw（本条目主来源）：`../../raw/ai-agents/agent-harness-anatomy-langchain.md`
- raw（对照来源）：`../../raw/ai-agents/agent-harness-anatomy.md`
- 原始 URL：https://www.langchain.com/blog/the-anatomy-of-an-agent-harness
- 作者：Vivek Trivedy（LangChain）
- 发布：2026-03-10
- 第三方核实：https://ghuntley.com/loop/（Ralph Loop 归因，2026-01-17）
- 抓取：2026-10-23
