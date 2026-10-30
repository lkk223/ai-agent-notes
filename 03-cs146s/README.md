
# 卷三导读

CS146S 的定位与 CS329Z 和 CMU 完全不同：**CS329Z 和 CMU 教你「造 Agent」，CS146S 教你「用 Agent」**。它面向的是日常用 Claude Code、Cursor 等工具写代码的工程师，教你怎么把 AI 编码工具用到极致。

课程分为四大模块：**基础**（W1-2：LLM 与 MCP）、**Agent 技艺**（W3-5：上下文工程、Agent 管理、终端）、**信任**（W6-7：安全与代码审查）、**交付**（W8-10：构建、运维、未来）。

2026 版课程围绕 MCP、Agent Skills、规格驱动开发（Spec-Driven Development）、循环工程（Loop Engineering）和软件工厂（Software Factory）重新组织，反映了行业的最新趋势。

> 💡 **讲师批注**：讲师批注：CS146S 的期末项目占成绩 80%，说明这门课的核心理念是「做中学」。如果你只看不练，等于没上这门课。


# 第 1 周　LLM 与 AI 编码基础（LLMs and AI Coding Foundations）


## W1.0 本周定位

第一周回答最基本的问题：**LLM 到底是什么？它为什么会犯错？怎么引导它产生稳定的输出？**这不是泛泛的「写好 prompt」指南，而是把 Prompt 工程当作一门科学方法来教。


## W1.1 LLM 的本质

CS146S 用一句话概括 LLM：**LLM 是一个根据上文预测下一个 Token 的概率系统**。它不是搜索引擎（不是在「查找」答案），不是数据库（不是在「检索」信息），而是在基于训练中学到的模式「生成」最合理的下一个 Token。

这解释了 LLM 的三个关键特性：**幻觉**（模型会生成看似合理但不正确的内容，因为它的目标是「合理」而非「正确」）、**不确定性**（同样输入可能产生不同输出）、**上下文依赖**（输出质量严重依赖你给的上下文）。


## W1.2 Prompt 工程即科学实验

CS146S 把 Prompt 工程当作实验科学：**形成假设 → 设计 Prompt → 观察结果 → 迭代**。不是「感觉这个 prompt 不错」，而是「这个 prompt 在 10 次测试中有 9 次产生了正确格式」。


## W1.3 作业：Prompt Playground

作业要求构建一个 Prompt 实验场：给定一个任务，设计多个 prompt 变体，每个变体跑 10 次，量化比较成功率。目标是把 Prompt 工程从「玄学」变成「可量化、可复现」的工程实践。


## W1.4 指定阅读

- Karpathy《Deep Dive into LLMs》——LLM 原理的深度讲解视频
- Google《Prompt Engineering Overview》——系统性 prompt 指南
- OpenAI《How OpenAI Uses Codex Internally》——内部实践


# 第 2 周　Coding Agent 解剖（Anatomy of a Coding Agent）


## W2.0 本周定位

如果说第 1 周教你「怎么跟 LLM 说话」，第 2 周教你「怎么给 LLM 装上手脚」。本周的核心是 **MCP（Model Context Protocol）**——Agent 工具接入的标准协议。


## W2.1 Agent 的四个核心组件

CS146S 把 Coding Agent 拆成四个组件：**感知**（理解任务和环境）、**规划**（分解步骤）、**执行**（调用工具）、**反馈**（评估结果）。这四个组件构成一个循环——与 CMU 的 Agent Loop 概念完全一致。


## W2.2 MCP 深入

MCP 是 AI 工具的「USB 接口」。核心概念：

- **MCP Server**：工具提供方。实现工具列表查询和工具执行两个接口。
- **MCP Client**：AI 应用方。发现 Server 提供的工具，发起调用。
- **通信协议**：标准 JSON-RPC，跨语言、跨平台。


## W2.3 作业：从零写 MCP Server

这是 CS146S 最重要的作业之一：**不用框架，从零手写一个 MCP Server**。实现工具列表握手和工具执行接口，然后让一个 AI 客户端连接并调用你的工具。

CS146S 嘉宾（Cognition 研究负责人）的原话：**「Once you've written the tool-listing handshake and watched a model decide (sometimes wrongly) which of your tools to call, the ecosystem stops being mysterious.」**——写完 MCP Server，Agent 生态系统对你就不再神秘。


## W2.4 指定阅读

- MCP Introduction / Server Examples / Authentication / SDK——MCP 官方文档
- 《APIs Don't Necessarily Make Good MCP Tools》——API 设计与 MCP 工具设计的区别


# 第 3 周　AI IDE 与上下文工程（AI IDEs and Context Engineering）


## W3.0 本周定位

CS146S 明确指出：**这是全课最重要的一周**。上下文工程正在取代 Prompt 工程成为核心竞争力——因为优化单个 prompt 已经到了天花板，真正决定 AI 代码质量的是你提供的上下文。


## W3.1 核心概念：Specs 即新源码

本周引用了一篇范式转移级的文章《Specs Are the New Source Code》：在 AI 编码时代，代码只是规格的「有损投影」。真正的源码是你的 Spec/PRD——Agent 根据规格生成代码，人审查规格而非逐行审查代码。


## W3.2 上下文失败的四种模式

《How Long Contexts Fail》一文揭示了四种上下文失败模式：

| 失败模式 | 表现 | 后果 |
| --- | --- | --- |
| 上下文中毒（Poisoning） | 错误信息被反复引用 | 模型基于错误前提推理 |
| 上下文干扰（Distraction） | 超过 100K Token 后模型倾向重复过去行为 | 忽略新指令 |
| 上下文混淆（Confusion） | 工具定义过多 | 模型不知道用哪个工具 |
| 上下文冲突（Conflict） | 上下文中有矛盾信息 | 性能下降 39% |


## W3.3 工具设计五原则

本周引用 Anthropic 的《Writing Effective Tools for Agents》，与 CMU 第 2 讲的原则一致但更加展开：精选不堆砌、命名空间分组、返回语义数据、优化 Token 效率、工具描述是性能杠杆。


## W3.4 作业

写一份设计文档（Design Doc），喂给 Agent，对比「有文档」和「没文档」两种情况下 Agent 产出的代码质量差异。这个实验让你直观感受上下文的价值。


## W3.5 嘉宾

Silas Alberti，Cognition（Devin）研究负责人。


# 第 4 周　Coding Agent 模式（Coding Agent Patterns）


## W4.0 本周定位

本周的核心问题：**Agent 应该有多大自主权？人在什么节点介入？**这是从「会用工具」到「会管理 Agent」的关键跃迁。


## W4.1 自主性光谱

CS146S 提出了 Agent 自主性的光谱：

| 任务复杂度 | 自主程度 | 人的角色 |
| --- | --- | --- |
| 简单任务（改个变量名） | 完全自主——不需要审查 | 抽查 |
| 中等任务（加个功能） | 80% 自主 + 人工润色 | 关键节点审查 |
| 复杂任务（重构架构） | 分阶段——每阶段需审批 | 深度参与 |


## W4.2 核心实践：检查点机制

检查点（Checkpoint）是在 Agent 工作流中设置的「暂停点」：Agent 到达检查点时暂停，展示当前进展和下一步计划，人类审批后继续。这让你能在 Agent 走偏时及时纠正，而不是等它跑完 50 步才发现方向错了。


## W4.3 作业

用 Claude Code 完成一个完整项目——目标不是「写了多少代码」，而是训练你成为「Agent 管理者」：你会设定目标、审查计划、在关键节点介入、验收结果。


## W4.4 嘉宾

Boris Cherny，Claude Code 的创造者。


# 第 5 周　现代终端（The Modern Terminal）


## W5.0 本周定位

AI IDE 解决「写代码」的问题，AI 终端解决「运行代码和管理系统」的问题。本周介绍 AI 增强的终端工具（以 Warp 为代表）。


## W5.1 核心概念

AI 终端的核心能力：自然语言转命令（「找出占用 8080 端口的进程并杀掉」）、命令结果解释、多步骤任务自动化。Claude Code 本身就运行在终端中——终端熟练度是 Agent 工作流的基础。


## W5.2 嘉宾

Zach Lloyd，Warp CEO。


# 第 6 周　AI 测试与安全（AI Testing and Security）


## W6.0 本周定位

CS146S 称之为「全课最硬核的一周」。当 AI 帮你写代码时，谁来保证代码安全？本周深入真实漏洞案例和安全工具。


## W6.1 真实案例：Copilot 提示注入 RCE

一个真实 CVE：攻击者在源码中植入恶意注释，Copilot 读取后修改了 VS Code 配置文件，开启了「YOLO 模式」（自动批准所有操作），然后执行了任意终端命令。这不是理论——是实际发生过的远程代码执行漏洞。


## W6.2 Semgrep 实战发现

Semgrep 团队在 11 个大型开源项目（800 万+行代码）上测试了 Claude Code 和 Codex 的漏洞发现能力：Claude Code 发现了 46 个真实漏洞（14% 正确率），但误报率 86%。更关键的是**非确定性问题**——同样的代码、同样的 AI，不同次运行结果不同。


## W6.3 作业

对你自己的 AI 生成代码运行 Semgrep 安全扫描，统计真实漏洞 vs 误报，修复所有高危问题。


## W6.4 嘉宾

Isaac Evans，Semgrep CEO。


# 第 7 周　代码审查与软件支持（Code Review and Software Support）


## W7.0 本周定位

本周继续安全主题，聚焦一个核心问题：**我们能在多大程度上信任 AI 生成的代码？**


## W7.1 AI 代码的「气味」

AI 生成的代码有独特的「气味」——表面看起来正确，但可能在边缘情况、安全处理、性能优化上有系统性盲区。审查 AI 代码不能简单复制审查人类代码的方法。


## W7.2 审查策略

- **对抗性审查**：让另一个 Agent 审查第一个 Agent 的代码——AI 审 AI。
- **边缘情况聚焦**：AI 代码最容易在边缘情况出错，重点审查异常处理、输入验证、并发安全。
- **不要假设 AI 是对的**：即使代码能运行、测试通过，也要检查是否有逻辑漏洞。


## W7.3 嘉宾

Tomas Reimers，Graphite CPO——分享了百万次 AI 代码审查的经验。


# 第 8–10 周　从原型到生产（Automated App Building / Post-Deployment / Future）


## W8：自动化应用构建

用 AI 从一句话生成完整 Web 应用（v0、Lovable 等工具）。但本周的真正教训是**原型与生产之间的鸿沟**——生成一个 Demo 很容易，但 Demo 到生产之间隔着测试、安全、性能、可维护性的深渊。

作业：用一句话生成一个应用，然后列出所有阻碍它上线的因素——这个清单才是真正的交付物。


## W9：部署后运维

系统部署不是终点。监控、告警、事故响应、自动排障——这些都在被 AI 重塑。Resolve AI 展示了用多 Agent 系统自动化 K8s 排障和事故响应的实践。


## W10：AI 软件工程的未来

收尾周由 a16z 合伙人 Martin Casado 分享行业趋势。核心观点：当 AI 能处理越来越多的实现工作时，人类开发者的核心价值在于**定义问题、审查方案、做出判断**——从「写代码的人」变成「Agent 的管理者」。


## CS146S 总结

CS146S 的 10 周可以浓缩为一条主线：**Vibe Coding（凭感觉用 AI 写代码）→ Agentic Engineering（系统性地管理 AI Agent 完成软件工程全流程）**。课程的核心不是教你用某个工具，而是教你一套方法论——工具会过时，方法论不会。
