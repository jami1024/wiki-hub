---
title: Code Mode 一年后的复盘：为什么胜出的是"让模型写代码"
topic: ai-agents
source:
  - ../../raw/ai-agents/next-token-ep07-after-code-mode.md
status: draft
updated_at: 2026-10-31
---

# Code Mode 一年后的复盘：为什么胜出的是"让模型写代码"

## 摘要

《The Next Token》第 7 期请来 Cloudflare 的 Matt Carey，与主持 Dillon、Sunil（同为 Cloudflare）、Rhys（Executor）一起复盘 Code Mode 从提出到被主流 agent 采纳的一年，并讨论"之后是什么"。他们的主线是：MCP 早期把全部工具 schema 塞进上下文，产生了"工具太多"的抱怨；真正的解法不是减少工具，而是改变工具的执行环境，让模型在一个可写代码的环境里自己搜索、组合、过滤结果。Code Mode 就是这个思路的命名版本（把 MCP 工具转成 TypeScript 类型，让模型写代码调用）。一年后 OpenCode 2.0、Pi 1.0、Codex、ChatGPT web 都已内置 JS 执行环境，Code Mode 从"显然该这么做但没人做"变成默认。节目的第二半转向生态后果：普通人可用的个人 agent、agent 与 agent 互相消费（"slop highway"）、以及 MCP 会不会被少数客户端变成新的应用商店。多个判断是发言者的预测而非已发生事实，文中已分别标注。

## 1. Code Mode 要解决的不是"工具太多"，而是执行环境

节目对 Code Mode 的定义由 Dillon 给出：本质是**脚本化工具调用**（scripted tool calling）——把 MCP server 的工具生成 TypeScript 类型交给模型，模型写一段 TS 去调用，就像人写脚本组合多个 API 一样。来源：00:12:52–00:14:22。

Matt 重述了问题起源：早期 MCP 的朴素做法是调 `list_tools` 拿回 JSON schema，塞进模型上下文，于是每个 MCP 工具都占用 context；上下文窗口当时不够长，40 个工具就开始挤爆，"MCP server 工具太多"因此成为被公开批评的点（GitHub MCP server 因 55 个工具承受了大量指责，Matt 认为它现在其实是实现最好的 server 之一）。来源：00:22:18–00:23:36。

但 Matt 的核心反驳针对的是后来的"bash 就够了"浪潮：给模型一个 bash 工具**并没有减少工具数量，而是换了工具的执行环境**。换成 bash 后自然获得三件事：渐进式发现（CLI 的 `--help`）、结果过滤（把 dev server 日志写文件再 grep）、以及把多个工具串起来。他也反过来指出 bash 的结构性弱点：CLI 的动作空间无法枚举（`--help` 没有标准格式）、凭据必须落到机器上、因此不该给后台 agent 破坏性工具。来源：00:15:08–00:17:15。

**归纳**：这条线索解释了为什么"MCP 太臃肿"和"CLI 才是未来"两种流行说法都只对了一半——争的是动作如何被表示和发现，而不是工具个数本身。

## 2. 源流：不是 Cloudflare 原创的想法，但他们命名并第一个做了客户端实现

Matt 交代的谱系比常见叙述更完整：

- 思想来源是 Apple 2021 年的 **CodeAct** 论文，以及 MCP 作者 **DSP** 在纽约向 **Kenton** 提出的"让客户端写代码来做 tool call"。Kenton 随后让 Matt 去实现。
- 博客由 **Sunil 与 Kenton** 撰写（节目时点约一年两个月前），也是 **Code Mode 这个名字**的来源；Matt 说他们是首批客户端侧实现，Claude Code 之后才落地。
- 命名之后有近一年"没人真的动手"，Matt 的解释是客户端要自带 REPL 或 JS 执行环境，成本高；很多 CLI 产品于是选择"我们已经有 bash"。
- **Pi（Earendil）是这条路线最鲜明的反例再转向**：节目说 Pi 早期只给 read/write/edit/bash，并公开表示不会接 MCP，后来做了完全相反的选择（"U-turn"）。这一转向可核实：Earendil 官方文章《“You Said No MCP!”》（2026-09-29）开头即承认 pi.dev 上曾有“Pi 不支持 MCP”的声明、以及团队在播客里多次贬低 MCP，并说明升级后的 Pi 已把 MCP 作为受支持功能。
- **OpenClaw 的 Peter** 走的是另一条路：MCPorter，把 MCP server 生成 SDK 再当 CLI 跑，坚持不用 MCP 客户端。

来源：00:18:34–00:21:30（CodeAct、DSP 与命名）、00:24:28–00:25:05（Pi 的转向与 MCPorter）。

## 3. Cloudflare 的实现关键：搜索不做索引，交给模型写代码

2026 年 2 月 20 日 Cloudflare 发布的 Code Mode MCP server（官方博客标题即 "give agents an entire API in 1,000 tokens"）只暴露两个工具 **search + execute**，把整份 API 压到约 1000 tokens，同时保留完整 API 访问。官方数字是 **2,500+ endpoints**（节目口述为“三千多个工具”）；博客另称该 server 遵循 OAuth 2.1、用 Workers OAuth Provider 把令牌限制到用户授权的权限。Matt 强调这里最容易误解的一点：**没有向量检索、没有排序算法、没有 k**。做法是把 OpenAPI spec 放进模型可访问的 REPL，让模型自己写代码在上面搜索（他举例 `.startsWith` 之类），并按需只把少量结果带回上下文。他的原话："It's just code."（他说别人通常要追问三次才相信）。

`execute` 则基本是一个带认证的 fetch。这个设计顺带解决了双份真相问题：不必同时维护 OpenAPI spec 与手写的 MCP 工具 schema（他提到 GitHub 那边是手写全部 schema，维护成本很痛），Cloudflare 的做法是每天重新加载 OpenAPI spec。Matt 自己的动机很直白：他在创业公司做过两年推荐系统，不想再碰"搜索质量评估"那套。来源：00:25:56–00:29:50。

## 4. 现状：执行环境的选型，以及"code mode 里套 code mode"的实测

到节目时点，Matt 与 Dillon 的判断是"所有主流 coding agent 都默认在做 code mode"：OpenCode 2.0 与 Pi 1.0 当天/近期发布并内置 JS 执行环境（经核实：OpenCode 2.0 博客发布于 2026-10-12，即录制当天，其 `@opencode/codemode` 包自带最小化 JavaScript 解释器，且 2.0 里所有 MCP server 默认 opt-in code mode；Pi v1.0.0 发布于 2026-10-01，v0.99.0 于 2026-09-29 引入 MCP 与 QuickJS sandbox 形态的 codemode），Codex 与 ChatGPT web 同样如此；Claude Code 的 tool search 已默认开启，但真正的 code mode 仍在实验 flag 后（Matt 说自己需要再确认）。Matt 用 React 作对照：2014 年写 React 到 2017/18 年成为多数开发者的日常，用了四五年，而 Code Mode 一年就完成了。来源：00:29:51–00:30:39、00:49:01–00:50:19。

执行环境的可选实现被逐一列出：**QuickJS**（Vercel 的 Run SDK，轻、可进 Wasm）、**V8 isolates / dynamic workers**（重）、**OpenCode 自写解释器**、**Callscript**（可序列化、支持暂停恢复，Rhys 用它做审批）、**Monty**（Pydantic 的 Python 版）、以及用 TS-in-Wasm 给生成代码做类型检查。来源：00:35:00–00:36:30、00:42:07–00:45:06。

关于"每个 server 都自带 code mode"的担忧，节目给出的经验是**实测更差**：外层闭包变量在内层不可用、返回大 PNG 超限、缺 R2 binding 后模型开始乱加 binding。但 Cloudflare 的折中方案（代码在 dynamic worker 里跑，`import container` 拿到容器能力，得到"三层 code mode"）被认为意外可用，前提是让模型清楚自己换到了另一个执行环境。Sunil 举 CF 内部 portal 的同类现象：明知是 code mode 嵌套，实际就是"写更多代码，然后能跑"。来源：00:38:45–00:40:53。

Rhys 同时给出两个与权限相关的判断：他想把 Executor 的审批系统拆掉，因为用户大多直接用 `--dangerously-skip-permissions` 或 auto 模式，审批只增加摩擦；以及**所有工具调用都应该是异步的**——Pi 的重构就把 tool call 改成默认异步，理由之一是能顺带用更便宜的批处理推理。来源：00:33:38–00:35:00、00:43:47–00:45:06。

**归纳**：这一节把"Code Mode 已普及"和"Code Mode 的正确形态还没定"区分开了。真正没解决的是权限、状态与暂停恢复，而不是生成代码本身——Matt 明确说难点不在 code mode，而在把工具接进 code mode 格式那一段（OpenAPI spec 不可靠、缺 header、安全 schema 缺失）。来源：00:45:49–00:46:35。

## 5. 对"之后是什么"的分歧：状态、位置与协议

四人对"next"的答案不同，这里按发言者分开记录，不合成统一结论：

- **Rhys（可复用与有状态）**：可复用脚本（Kent C. Dodds 的 Kody 让 agent 写脚本再 import 进 code mode，兼作一种记忆）、triggers / MCP events（服务回调 harness）、durable execution（例：让 Gmail 延迟五分钟发送，留出撤销窗口）。来源：00:50:19–00:52:37。
- **Matt（unikernel）**：为任务即时构造内核，客户端通过 MCP 直连；他认为"所有东西都跑在通用 Linux VM 上"不可持续。来源：00:53:27–00:55:00。
- **Dillon（执行位置）**：两三年内 Code Mode 会移出客户端，动态放置到离被调用 API 更近的地方，因为推理变快后工具往返的 100ms 变得显著。来源：00:59:54–01:00:44。
- **共识（Cap'n Web）**：Kenton 的 Cap'n Web 用 stub + promise pipelining 同时解决计算局部性、请求瀑布和链式调用，且设计早于 MCP（Matt 说它源自 Kenton 对 GraphQL 的不满）。Sunil 提议把它作为 MCP 的底层协议，Matt 称这是他见过最贴合的低层协议。来源：00:57:31–00:59:18、01:00:44–01:01:28。
- **Sunil（MCP 收敛为资源）**：MCP 可能只剩一个原语——resource（URI + MIME type）。skill 就是一份 markdown 资源，tool 就是某个 URI 的执行，还可以返回 stub 让客户端调用；好处是终于有了**工具的分层结构**（类似 sitemap），skill 与 API 可以放在一起。他同时预测 MCP 会变成中间件，框架里一行 `mcp: true`。来源：00:55:49–00:59:18（资源原语与分层结构）、01:20:44–01:21:19（`mcp: true` 预测）。

## 6. 生态后果：普通人、slop highway 与准入权

节目后半段从技术转向使用侧，主要谈三件事。

**个人 agent 的临界点是"接进通讯渠道"。** Dillon 的妻子 Justine 一上手就给 GrokBot 排任务（联系租车公司、每天早上查天气发短信）；Sunil 认为真正解锁的是能发邮件、打电话、用即时通讯，同时这也让风险变大。Rhys 弃用 GrokBot 的理由是"那不是我的电脑"；Matt 用 Instinct（在 WhatsApp 里）找房、并让它向葡萄牙电信运营商追讨退款，落款统一加 "Sent from Matt's agent"，最后自动升级成向监管机构的正式投诉。经济性被质疑：Sunil 认为 $20/月撑不住用户狂开 sub-agent 的推理与 CPU 成本。来源：01:03:30–01:11:00。

**"slop highway"（Sunil 造的词）。** 证据主要是 agent 对 agent 的互相消费：GitHub 上 agent 互相回复、maintainer 的 PR 下出现五六个"免费 review"；Sunil 把坏台灯挂上 Facebook Marketplace，立刻收到二十条"我真的很想要这个坏台灯"，全部来自别人的 agent。他预测会**出现反制**（平台封禁、rate limit），并把 Muse 的 MCP 准入门槛作为正面例子：要让 server 进 Muse，其能力必须覆盖网站全部功能、不能把用户导回网页——效果是"把互联网 headless 化"。来源：01:11:06–01:17:17。

**准入权之争。** 隐忧是 Figma 一类做法：MCP server 只对特定客户端（Claude、Codex）开放，Matt 认为这会把开放网络变成应用商店。传输层上他们反对引入 gRPC 分支（提到某大型云厂商力推），理由是会造成客户端与服务端的裂裂，主张统一走 Streamable HTTP。来源：01:16:41–01:18:36。

## 边界与待核实事项

本文是节目转录的整理，不是可复现的技术评测，多个关键判断属于**发言者断言或预测**：

- **未能核实**：“所有主流 harness 都实现了 code mode”与“工具从三千个压到约 1000 tokens”在节目里无对照实验；Claude Code 是否已上 code mode 本条目未能确认——Matt 本人在节目中也说要再确认，实际查证时 Anthropic 文档在本地地区被阻断，二手资料又互相矛盾（一说 tool search 已默认，一说需用 `ENABLE_TOOL_SEARCH` 环境变量开启），因此本条保留为“存疑”。
- 时间线含混：转录中出现 "April last year"、"summer of last year" 与 "February last year"，但同时把 2026 年 2 月的发布称为"今年二月"，相对年份表述不一致，本条目未强行换算成具体日期。
- 节目内自述日期为 2026 年 10 月 12 日；产品与型号（OpenCode 2.0、Pi 1.0、Haiku 5.5、Grok 4.6、Muse、Instinct、OpenClaw、Jev）均按 ASR 原样保留，未独立核实其版本或能力。
- 结尾的 Lockheed Martin 轶事与"MCP Dev Summit 展位对话"来自单一叙述者，无法交叉验证。
- raw 为 YouTube 自动字幕（en-US），未人工校对，专有名词与数字有误识风险；本条目凡引用英文原话，均取自该自动字幕，不保证逐字准确。
- 正文中的 `hh:mm:ss` 定位已把 raw 重制为**带时间戳字幕稿**（每段前缀该段起止时间，单句精度约 ±3 秒）；此前版本无时间戳，locator 只能靠段落关键词。结尾 Lockheed Martin 轶事见 01:23:25–01:24:12。

## 已核实的外部事实（2026-10-31 查）

只列出本条目依赖、且已读到官方页面的项；未列入者仍按上文“待核实”处理。

| 节目中的说法 | 核实结果 | 出处 |
| --- | --- | --- |
| Cloudflare 在“今年二月”发布 code mode MCP server，两个工具、约 1000 tokens | 博客发布于 2026-02-20；官方数字为 2,500+ endpoints 覆盖、约 1,000 tokens、OAuth 2.1 | [Code Mode: give agents an entire API in 1,000 tokens](https://blog.cloudflare.com/code-mode-mcp/) |
| OpenCode 2.0 “今天”发布并走 code mode | 博客日期 2026-10-12；`@opencode/codemode` 自带最小 JS 解释器，2.0 中 MCP server 默认 opt-in code mode | [OpenCode 2.0](https://anoma.ly/notes/opencode-2-0) |
| Pi 1.0 刚发布，且此前公开拒绝 MCP | v1.0.0 发布于 2026-10-01；v0.99.0（2026-09-29）引入 MCP 与 codemode（QuickJS sandbox）；官方文章承认此前“不支持 MCP”的公开立场 | [Pi v1.0.0 release](https://github.com/earendil-works/pi/releases/tag/v1.0.0)、[“You Said No MCP!”](https://earendil.com/posts/you-said-no-mcp/) |

仍未核实：Claude Code 的 code mode/tool search 当前默认状态；Muse、Instinct、GrokBot、Haiku 5.5、Grok 4.6 等产品的版本与能力；Lockheed Martin 轶事。

## 关联条目

- [Agent Harness 解剖：模型之外的工程才是产品](agent-harness-anatomy.md)：讨论 harness 如何组织工具、上下文与反馈循环，并提醒"工具范围按当前步骤暴露"；本期补充了这条建议在实践中如何被改写——用 Code Mode 让模型自行搜索和组合工具，而对"工具多少"的争论也在这期里被重新解释为执行环境问题。
- [Agent 后端的三层架构：从循环到可维护系统](agent-backend-architecture.md)：给出工具调用与模型审核的具体编排示例；本期从另一侧讨论同一问题，即审批层的现实摩擦（用户普遍绕过）与"所有工具调用异步化"的取舍，两者可以对照阅读，但本期的权限结论是经验性主张，未经安全评测。

## 来源

- 原始视频：[Ep 07 — "What's after Code Mode"（The Next Token）](https://www.youtube.com/watch?v=0hRlvn0IPMI)
- 主持：Dillon Mulroy、Rhys、Sunil；嘉宾：Matt Carey（Cloudflare）
- 发布日期：未提供（节目内自述 2026 年 10 月 12 日）
- [raw：带时间戳的 YouTube 自动字幕稿及处理说明](../../raw/ai-agents/next-token-ep07-after-code-mode.md)
- 外部核实（2026-10-31）：[Cloudflare Code Mode 博客](https://blog.cloudflare.com/code-mode-mcp/)、[OpenCode 2.0](https://anoma.ly/notes/opencode-2-0)、[Pi v1.0.0 release](https://github.com/earendil-works/pi/releases/tag/v1.0.0)、[Earendil “You Said No MCP!”](https://earendil.com/posts/you-said-no-mcp/)
- 材料获取日期：2026-10-31；视频仅取字幕，未下载音视频。
