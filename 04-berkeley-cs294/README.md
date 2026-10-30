
# 卷四导读

Berkeley CS294-196 与前三门课的形式完全不同：它不是「一位老师系统讲授」，而是**每周邀请一位行业顶级研究者做客座讲座**。2024 秋季班的讲师阵容包括 Denny Zhou（Google DeepMind）、Shunyu Yao（ReAct 作者）、Chi Wang（AutoGen）、Omar Khattab（DSPy）、Graham Neubig（CMU）、Jim Fan（NVIDIA）、Percy Liang（Stanford）、Dawn Song（Berkeley）等。

这门课的价值在于**前沿视野**——你能直接听到各领域的开创者讲解他们的工作。本卷精选与 Agent 工程最相关的讲座做笔记。


# 2024 秋季班精选


# 讲座 1　LLM 推理（Denny Zhou, Google DeepMind）


## 核心内容

Denny Zhou 是 Chain-of-Thought（思维链）论文的第一作者。本场讲座深入讲解 LLM 推理能力的前沿：

- **Chain-of-Thought 不需要 Prompt**：最新研究表明，即使不特意提示「让我们一步步思考」，模型在解码时也能自发产生推理链——关键在于解码策略。
- **LLM 还不能自我纠正推理**：模型很难在没有外部反馈的情况下发现自己的推理错误——这对 Agent 的 Reflection 设计有重要启示。
- **前提顺序影响推理**：改变前提的呈现顺序会显著影响推理结果——上下文工程中信息排序至关重要。
- **CoT 使 Transformer 能解决串行问题**：思维链让模型从「并行计算」变成「串行推理」，突破了 Transformer 的表达能力上限。

> 💡 **讲师批注**：讲师批注：「LLM 不能自我纠正」这个发现对 Agent 设计影响很大——不要指望 Agent 自己能发现所有错误，外部验证（测试、规则检查、另一个 Agent 审查）是必需的。


# 讲座 2　LLM Agent 简史与概述（Shunyu Yao, OpenAI）


## 核心内容

Shunyu Yao 是 ReAct 论文的第一作者，也是 OpenAI 的 Agent 研究员。本场讲座是 Agent 领域的「通史」：

- **ReAct 的诞生**：ReAct 的核心洞察是「推理和行动应该交织进行」——纯推理（CoT）容易脱离实际，纯行动（Act）缺乏规划。两者交替让 Agent 既能思考又能验证。
- **WebShop**：一个模拟网购环境，用于训练和评估 Web Agent。Agent 需要在网页上搜索、筛选、比较、购买商品。
- **从 ReAct 到现代 Agent**：ReAct 的「推理→行动→观察」循环是所有现代 Agent 的基础模式。


# 讲座 3　Agent 框架与 AutoGen（Chi Wang）+ 多模态知识助手（Jerry Liu）


## 核心内容

**AutoGen**（Chi Wang, Microsoft）是多智能体对话框架的先驱：让多个 Agent 通过自然语言对话来协作完成任务。核心创新是「可对话的 Agent」——Agent 之间不是通过 API 调用通信，而是通过自然语言讨论。

**StateFlow**：在 AutoGen 基础上加入状态机——用状态驱动的工作流增强多 Agent 协作的可控性。

**LlamaIndex**（Jerry Liu）是 RAG 系统的标准框架——提供了从数据加载、索引、检索到生成的完整管线。


# 讲座 5　Compound AI 系统与 DSPy（Omar Khattab, Databricks）


## 核心内容

Omar Khattab 是 DSPy 的创造者。本场讲座阐述了 Compound AI 系统的愿景：

- **从模型到系统**：AI 应用的未来不是「更大的模型」，而是「更好的系统设计」——多个小组件（检索器、生成器、验证器）的精妙组合。
- **DSPy 的核心理念**：把 Prompt 工程变成软件工程——声明输入输出（Signature），框架自动优化 Prompt。
- **微调与 Prompt 优化结合**：两者不是对立关系——先用 DSPy 优化 Prompt（快速迭代），再用优化后的系统生成训练数据做微调（长期固化）。


# 讲座 6　Coding Agent（Graham Neubig, CMU）


## 核心内容

这场讲座与 CMU 11-768 第 6 讲内容互补，更侧重 OpenHands 平台的设计哲学：

- **SWE-agent 的 ACI 设计**：为 LLM 专门设计的计算机接口——不是简单的终端，而是结构化的文件浏览、精确编辑、智能搜索。
- **OpenHands 的愿景**：做一个开源的、可扩展的 Agent 开发平台，让研究者能方便地实验不同的 Agent 设计。


# 讲座 9　GR00T：通用机器人蓝图（Jim Fan, NVIDIA）


## 核心内容

Jim Fan 是 NVIDIA 的具身智能研究负责人。本场讲座把 Agent 概念从数字世界扩展到物理世界：

- **Voyager**：Minecraft 中的自主探索 Agent——通过 LLM 生成代码来控制游戏角色，自主探索、学习技能、建造工具。
- **Eureka**：用 LLM 自动生成奖励函数来训练机器人——人类只需描述任务目标，LLM 自动设计奖励信号。
- **DrEureka**：从仿真到现实的迁移——在仿真中训练机器人策略，然后部署到真实机器人。

> 💡 **讲师批注**：讲师批注：虽然机器人 Agent 与软件 Agent 的应用场景不同，但底层的设计模式（感知→规划→执行→反馈）完全一致。Jim Fan 的工作展示了 Agent 范式的通用性。


# 讲座 11　Agent 能力评测与 Anthropic RSP（Ben Mann, Anthropic）


## 核心内容

- **负责任扩展政策（RSP）**：Anthropic 的 AI 安全框架——随着模型能力增强，逐步提高安全要求。
- **Computer Use 模型**：让模型直接操作电脑界面（看屏幕截图、移动鼠标、点击、输入）——这是 GUI Agent 的技术基础。


# 讲座 12　安全可信的 AI Agent（Dawn Song, UC Berkeley）


## 核心内容

Dawn Song 是 AI 安全领域的顶级学者。本场讲座覆盖了 Agent 安全的多个维度：

- **DecodingTrust**：全面评估 LLM 可信度的框架——包括鲁棒性、隐私、公平性、可解释性。
- **表示工程（Representation Engineering）**：通过分析模型内部表示来理解和控制模型行为——比 Prompt 更深层的干预。
- **训练数据提取攻击**：攻击者可以从模型中提取训练数据的片段——Agent 的隐私风险不仅来自工具调用，也来自模型本身。


# 2025 秋季班精选


# 讲座：后训练可验证 Agent（Jiantao Jiao, NVIDIA）


## 核心内容

- **SWE-bench Verified**：OpenAI 人工验证的 SWE-bench 子集——排除了描述不清的 Issue，使评测更可靠。
- **BrowseComp**：浏览 Agent 的评测基准——测试 Agent 在网上搜索和整合信息的能力。


# 讲座：多智能体 AI（Noam Brown, OpenAI）


## 核心内容

Noam Brown 是多智能体系统的顶级研究者（Libratus、Pluribus 等扑克 AI 的创造者）。他分享了多智能体系统在 LLM 时代的新机遇和挑战。


# 讲座：AI Agent 自动化科学发现（James Zou, Stanford）


## 核心内容

- **Virtual Lab**：多个 AI Agent 组成虚拟实验室，自主设计并验证了新的 SARS-CoV-2 纳米抗体。
- **Paper2Agent**：把研究论文变成可交互的 AI Agent——读者可以直接与论文「对话」，让 Agent 复现实验。


# 讲座：真实世界 Agent 部署经验（Clay Bavor, Sierra）


## 核心内容

- **τ2-Bench**：双控制环境下的对话 Agent 评测——Agent 和用户都可以操作环境，测试协作能力。
- **Voice Sims**：在 Agent 与真实客户对话之前，先在模拟环境中测试——生产级 Agent 的安全实践。
