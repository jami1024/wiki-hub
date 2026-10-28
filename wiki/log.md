# Wiki Log

| 日期 | 操作 | 条目 | 来源 | 备注 |
|---|---|---|---|---|
| 2026-09-24 | 初始化 | — | — | 创建 Wiki 数据目录，尚未导入来源 |
| 2026-10-23 | ingest | [ai-agents/agent-harness-anatomy](ai-agents/agent-harness-anatomy.md) | https://x.com/akshay_pachaar/article/2041146899319971922 | 新增 raw 全文 + wiki 条目；x.com 直连不可达，正文经第三方 clipping 与 link-vault 转录交叉核对；原文“12 项组件”实列 11 项，已在条目内标注 |
| 2026-10-23 | ingest | ai-agents/agent-harness-langchain（已合并） | https://www.langchain.com/blog/the-anatomy-of-an-agent-harness | 新增：LangChain 原文（2026-03-10，Vivek Trivedy）+ 与 Akshay 篇的逐维对照；核实 Ralph Loop 归属冲突（ghuntley.com/loop） |
| 2026-10-23 | update | [ai-agents/agent-harness-anatomy](ai-agents/agent-harness-anatomy.md) | https://x.com/akshay_pachaar/article/2041146899319971922 | 补“源流与对照”小节 + 关联条目互链 + Ralph 归因冲突指引；核心事实未改 |
| 2026-10-23 | ingest | [ai-agents/agent-backend-architecture](ai-agents/agent-backend-architecture.md) | https://www.bilibili.com/video/BV1winH6DEgt/ | 视频无字幕轨，音频经 whisper large-v3-turbo 转写（135 段/4800 字）；raw 保留 ASR 原文 + 同音误识对照表 |
| 2026-10-23 | merge | [ai-agents/agent-harness-anatomy](ai-agents/agent-harness-anatomy.md) | — | 将 agent-harness-langchain 的 12 维对照表、TerminalBench 补充事实与 Ralph 归因冲突并入本条，删除独立条目；两篇 raw 均保留，对照来源写入 frontmatter |
| 2026-10-23 | update | [ai-agents/agent-harness-anatomy](ai-agents/agent-harness-anatomy.md) | — | 按用户要求移除「与 LangChain 原文的对照」整节及 frontmatter 第二来源；Ralph Loop 归因疑点缩为一条内联备注；raw 文件保留未删 |
| 2026-10-23 | cleanup | — | https://www.langchain.com/blog/the-anatomy-of-an-agent-harness | 删除已无条目引用的 raw/ai-agents/agent-harness-anatomy-langchain.md；原页可重新抓取 |
| 2026-10-27 | update | [ai-agents/agent-backend-architecture](ai-agents/agent-backend-architecture.md) | https://www.bilibili.com/video/BV1winH6DEgt/ | 按 library 0.2.0 重读既有 raw 并原路径重写：解释分层动机、接口与测试替身、审核反馈和组装；修正将示例模型审核泛化为通用要求的表述；保留时间戳与 ASR 边界；raw 未改，索引同步更新 |
| 2026-10-27 | update | [ai-agents/agent-harness-anatomy](ai-agents/agent-harness-anatomy.md) | https://x.com/akshay_pachaar/article/2041146899319971922 | 按 library 0.2.0 重读既有 raw 并原路径重写：围绕反馈循环、上下文、恢复验证与复杂度取舍组织正文；合并重复清单和数字表，区分作者转述、归纳及待核实事项；raw 未改，索引同步更新；本次未新增外部事实核查 |
