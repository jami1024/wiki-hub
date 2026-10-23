---
title: Agent Harness 解剖：模型之外的工程才是产品
topic: ai-agents
source:
  - ../../raw/ai-agents/agent-harness-anatomy.md
  - ../../raw/ai-agents/agent-harness-anatomy-langchain.md
status: draft
updated_at: 2026-10-23
---

# Agent Harness 解剖：模型之外的工程才是产品

## 一句话摘要

Akshay Pachaar 汇总 Anthropic、OpenAI、LangChain、CrewAI、AutoGen 等实现，论证“agent = 模型 + harness”，并给出生产级 harness 的组件清单、单轮循环的七个步骤、以及每个架构师必须做的七个取舍。

## 与 LangChain 原文的对照

本文（2026-04-06）晚于 Vivek Trivedy 在 LangChain 博客的同名文章（2026-03-10）27 天，并直接引用其「If you're not the model, you're the harness」与 TerminalBench「Top 30 → Top 5」实验。Akshay 的增量在跨厂商横向汇总，LangChain 的增量在从模型缺口纵向推导每个 primitive 为何存在。

| 维度 | LangChain / Vivek Trivedy（2026-03-10） | Akshay Pachaar（2026-04-06） |
| --- | --- | --- |
| 组织方法 | 从模型缺口**反推**（behavior → design），公理式 | 跨厂商**横向汇总**，清单式 |
| 组件划分 | 不设固定数量，六组 primitive | 宣称 12 项、实列 11 项 |
| filesystem 地位 | **最基础 primitive**，独立成节 | 未单列；状态管理讲 typed dict / git checkpoint |
| sandbox | 独立成节：安全 + 按需扩缩 + allow-list/网络隔离 | 仅一句「在沙箱环境执行」 |
| bash 作为通用工具 | 独立成节，「给模型一台计算机」 | 未成节（Tools 一节讲 schema 注册与校验） |
| skills | 作为 context rot 对策，progressive disclosure | **完全未提** |
| hooks/middleware | 独立的确定性执行层（compaction、continuation、lint） | 无独立层，散布在各组件描述中 |
| 记忆实现锚点 | AGENTS.md 文件标准 + Web Search/MCP | CLAUDE.md/MEMORY.md、LangGraph Store、OpenAI Sessions、三级记忆 |
| ReAct 的称呼 | ReAct loop | TAO（Thought-Action-Observation） |
| 共演化证据 | 一手可核验：Codex-5.3 prompting guide 的 apply_patch、TerminalBench leaderboard | 一句断言，无出处 |
| 生产策略细节 | 少（偏概念推导） | 多（Stripe 重试上限 2 次、~40 个工具门控、1k–2k token 子 agent 摘要） |
| 反向结论 | 不必沿用模型 post-train 时的 harness | 未强调；主线是脚手架会被拆除 |

*归纳*：两份清单不冲突，是不同切面。LangChain 切「模型缺什么 → 补什么 primitive」，Akshay 切「一条请求经过哪些环节」。前者解释为什么有这些部件，后者解释怎么串起来跑一轮。

**TerminalBench 的补充事实**：LangChain 补了一个 Akshay 没写的点——Opus 4.6 在 Claude Code 里的分数远低于同一个 Opus 4.6 配其他 harness，直接反驳「官方 harness 最优」。

**Ralph Loop 归因冲突**：Akshay 写「Anthropic developed a two-phase "Ralph Loop" pattern」，链接 `anthropic.com/research/long-running-Claude`；LangChain 链到 `ghuntley.com/loop/`。后者已核实：Huntley 自引「the original ralph post」，把 Ralph 定义为单仓库、单进程、每轮一个任务的 orchestrator 模式，页面发布 2026-01-17。判断：Ralph 概念源自 Huntley 的可能性更大，Anthropic 更可能是把它做成两阶段实现；引用 Akshay 时应改为「Anthropic 采用的 Ralph 变体」。

## 核心判断

1. **失败的 agent 首先要怀疑 harness，而不是模型。** 原文开篇即把 demo 到生产之间的塌方归因于模型外围：「The problem isn't your model. It's everything around your model.」定位：*正文第 3 段*。
2. **harness 有定义边界，且被官方文档采用。** Anthropic 称 SDK 是「the agent harness that powers Claude Code」；OpenAI Codex 团队把 "agent" 与 "harness" 等同，指非模型基础设施。定位：*What Is the Agent Harness? 第 1 段*。
3. **agent 是涌现行为，harness 是产生该行为的机器。** 「When someone says "I built an agent," they mean they built a harness and pointed it at a model.」定位：*What Is the Agent Harness? 第 3 段*。
4. **harness 对应计算机体系结构，而非“提示词包装”。** 借 Beren Millidge 的类比：context window 是 RAM、外部数据库是磁盘、工具是设备驱动、**harness 是操作系统**，结论是「We have reinvented the Von Neumann architecture」。定位：*What Is the Agent Harness? 第 4 段*。
5. **三层工程是包含关系，不是并列。** prompt engineering 管指令、context engineering 管“模型看到什么、何时看到”、harness engineering 涵盖两者加全部应用基础设施。「The harness is not a wrapper around a prompt.」定位：*Three Levels of Engineering*。
6. **换 harness 不换模型，排名可以翻天。** 同权重同模型，LangChain 只改外围基础设施就「jumped from outside the top 30 to rank 5 on TerminalBench 2.0」；另有研究让 LLM 自行优化该基础设施，达到 76.4% 通过率，超过人工设计系统。定位：*正文第 2 段*；同类结论复述于 *The Harness Is the Product* 第 1 段（"moved agents by 20+ ranking positions"）。
7. **上下文是稀缺资源，且位置比长度更致命。** context rot：关键内容落在窗口中部时性能下降 30%+（Chroma 研究，与 Stanford "Lost in the Middle" 互证）；即百万 token 窗口也逃不过指令遵循衰减。定位：*Context Management* 第 1 段。
8. **错误会复利，所以错误处理是架构问题。** 「a 10-step process with 99% per-step success still has only ~90.4% end-to-end success」。定位：*Error Handling* 第 1 段。
9. **验证回路是玩具与生产的分界线。** 三类手段：规则反馈（测试/lint/类型检查）、视觉反馈（Playwright 截图）、LLM-as-judge；Claude Code 作者 Boris Cherny 称「giving the model a way to verify its work improves quality by 2 to 3x」。定位：*Verification Loops*。
10. **先最大化单 agent，再谈多 agent。** Anthropic 与 OpenAI 共同建议；拆分门槛是工具过载超过 ~10 个重叠工具，或任务域明显分离。定位：*Seven Decisions* 第 1 条。
11. **harness 应随模型变强而变薄（脚手架隐喻）。** 建筑脚手架在楼建成后拆除；Manus 六个月内重写五次，每次都在删复杂度：「Complex tool definitions became general shell execution. "Management agents" became simple structured handoffs.」定位：*The Scaffolding Metaphor* 第 2 段。
12. **设计好坏有一个可证伪的测试。** 「if performance scales up with more powerful models without adding harness complexity, the design is sound.」定位：*The Scaffolding Metaphor* 末段（原文称 "future-proofing test"）。*归纳*：这条其实等价于“harness 不应替代模型能力，只应补模型缺口”。
13. **harness 不是会商品化的中间层。** 「The harness is not a solved problem or a commodity layer. It's where the hard engineering lives.」定位：*The Harness Is the Product* 第 2 段。

## 证据与关键数字

| 数字 | 含义 | 定位 |
| --- | --- | --- |
| top 30 外 → 第 5 | LangChain 仅改 harness 后的 TerminalBench 2.0 排名 | 正文第 2 段 |
| 76.4% | LLM 自优化 harness 的通过率（超过人工设计） | 正文第 2 段 |
| 30%+ | 关键内容位于窗口中部时的性能下降 | Context Management |
| ~150 字符 | Claude Code 记忆索引每条体积（始终加载） | Memory |
| 1,000–2,000 token | 每个 subagent 只回传的浓缩摘要长度 | Context Management |
| 32 KiB | OpenAI AGENTS.md 级联文件的大小上限 | Prompt Construction |
| 6 类 | Claude Code 工具分类（文件/搜索/执行/网络/代码智能/subagent） | Tools |
| ~40 个 | Claude Code 独立门控的离散工具能力 | Guardrails and Safety |
| 99% × 10 步 → ~90.4% | 错误复利 | Error Handling |
| 2 次 | Stripe 生产 harness 的重试上限 | Error Handling |
| 2–3x | 给模型验证手段带来的质量提升（Boris Cherny） | Verification Loops |
| 3.6x | LLMCompiler 相对顺序 ReAct 的加速 | Seven Decisions 第 2 条 |
| 26–54% / 95%+ | ACON：token 降幅与保持的准确率（优先保留推理轨迹而非原始工具输出） | Seven Decisions 第 3 条 |
| 80% | Vercel 从 v0 移除的工具比例，效果反而更好 | Seven Decisions 第 6 条 |
| 95% | Claude Code 靠懒加载实现的上下文缩减 | Seven Decisions 第 6 条 |
| 5 次 / 6 个月 | Manus 的重写次数与周期 | The Scaffolding Metaphor |

## 概念解释

### harness 的组成（原文称 12 项，实列 11 项）

1. **Orchestration Loop** — TAO（Thought-Action-Observation，即 ReAct）循环；机械上常只是一个 while 循环，「The complexity lives in everything the loop manages」。Anthropic 自述为 "dumb loop"，智能全在模型里。
2. **Tools** — 以 schema 注入上下文；工具层负责注册、校验、参数抽取、沙箱执行、结果格式化。
3. **Memory** — 短期（会话内）与长期（跨会话）；Claude Code 三级结构：轻量索引（始终加载）→ 按需拉取的主题文件 → 仅可搜索的原始 transcript。关键设计原则：**把自身记忆当 "hint"，行动前先对真实状态核验**。
4. **Context Management** — 五种生产策略：compaction、observation masking（JetBrains Junie 隐藏旧工具输出但保留调用可见）、just-in-time 检索（grep/glob/head/tail 而非整文件）、subagent 委派、结构化笔记。目标是「smallest possible set of high-signal tokens」。
5. **Prompt Construction** — 分层拼装；OpenAI Codex 用严格优先级栈：服务端 system message > 工具定义 > developer 指令 > user 指令（级联 AGENTS.md）> 会话历史。
6. **Output Parsing** — 依赖原生 tool calling 返回结构化 `tool_calls`；有 tool call 就执行并继续，没有即最终答案。
7. **State Management** — LangGraph 用类型化字典 + reducer 在 super-step 边界 checkpoint（支持恢复与时间旅行调试）；Claude Code 用 git commit 当检查点、progress 文件当结构化草稿纸。
8. **Error Handling** — LangGraph 区分四类错误：transient（退避重试）、LLM-recoverable（作为 ToolMessage 回喂让模型调整）、user-fixable（中断交人）、unexpected（上抛调试）。
9. **Guardrails and Safety** — OpenAI SDK 三层：input / output / tool guardrail，触发 "tripwire" 立即停机。Anthropic 在架构上把权限执行与模型推理分离：「The model decides what to attempt; the tool system decides what's allowed.」
10. **Verification Loops** — 见核心判断 9。
11. **Subagent Orchestration** — Claude Code 三种执行模型：Fork（父上下文逐字节副本）、Teammate（独立终端面板 + 文件邮箱）、Worktree（独立 git worktree 与分支）；OpenAI SDK 则是 agents-as-tools 与 handoffs 两种。

### 单轮循环的七个步骤

Prompt Assembly → LLM Inference → Output Classification → Tool Execution（只读可并发、写操作串行）→ Result Packaging（错误也作为结果回喂）→ Context Update（接近上限触发 compaction）→ Loop。

终止条件是分层的：无 tool call、超最大轮数、token 预算耗尽、guardrail tripwire、用户中断、安全拒绝。

### 跨上下文窗口的长任务

Anthropic 的 "Ralph Loop" 两阶段：**Initializer Agent** 先搭环境（init 脚本、progress 文件、feature 列表、首次 git commit），之后每个会话的 **Coding Agent** 读 git log 与 progress 文件自我定位、挑最高优先级的未完成 feature、干活、提交、写总结。**文件系统提供跨窗口的连续性。**

### 七个架构取舍

单 agent vs 多 agent｜ReAct vs plan-and-execute｜上下文管理策略（五选一）｜验证回路设计（计算式 vs 推断式，即 Martin Fowler 团队的 guides/sensors）｜权限宽松 vs 严格｜工具范围（越少越好）｜harness 厚度（Anthropic 押薄 harness + 模型进步，图框架押显式控制）。

### 共演化（co-evolution）

模型现在是在特定 harness 在环的情况下 post-train 的。Claude Code 的模型学会了配合它被训练时的那套 harness，「Changing tool implementations can degrade performance because of this tight coupling.」*归纳*：这意味着换 harness 不是免费的重构，存在训练时的耦合成本。

## 关联条目

- [Agent 后端的三层架构：从循环到可维护系统](agent-backend-architecture.md) — 实现层：把同一个循环拆成领域/应用/适配三层，并用另一次模型调用审核工具调用。
- [Agent Harness 解剖：模型之外的工程才是产品](agent-harness-anatomy.md) — 同一来源的整理，概念上与本条互补。
- 尚未入库、仅搜索所见（未核实）：Addy Osmani `addyosmani.com/blog/agent-harness-engineering/`；Arize、Snowflake、puppygraph 的 "what is an agent harness" 术语条目。

## 未解决问题

- **原文自相矛盾**：小节标题写 "The 12 Components"，实际只列 11 项。第 12 项是什么，原文未给，本条目不做补写。
- 文中 "The term was formalized in early 2026" 未给出形式化来源（谁、在哪篇文献里定义）。
- 76.4% 通过率的"separate research project"未具名，无法回溯。
- TerminalBench 2.0 的排名变化（top 30 外 → 第 5）未给评测快照或日期。
- ACON、LLMCompiler 等研究仅有一句结论，未附论文链接。
- 原文关于 Claude Code 内部机制（~40 个工具能力、三级记忆、Ralph Loop）属二手转述，未标注 Anthropic 官方出处，引用时应回溯一手文档。
- Ralph Loop 被归给 Anthropic，与 LangChain 篇归给 Geoffrey Huntley 冲突；证据与判断见上文「与 LangChain 原文的对照」末段。

## 来源

- raw：`../../raw/ai-agents/agent-harness-anatomy.md`（正文全文 + 获取途径与清洗说明）
- raw（对照来源）：`../../raw/ai-agents/agent-harness-anatomy-langchain.md`
- 原始 URL：https://x.com/akshay_pachaar/article/2041146899319971922
- 对照 URL：https://www.langchain.com/blog/the-anatomy-of-an-agent-harness（Vivek Trivedy，2026-03-10）
- 第三方核实：https://ghuntley.com/loop/（Ralph Loop 归因，2026-01-17）
- 作者：Akshay Pachaar (@akshay_pachaar)
- 发布：2026-04-06T13:31:58Z（据 X status API 时间戳与转录页元数据）
- 抓取：2026-10-23
