
# 第 5 讲　框架与编排（Frameworks & Orchestration）


## 5.0 本讲定位

前几讲你学会了 Agent 的核心概念——循环、工具、RAG。这一讲讲「怎么把这些概念组织成可维护的工程系统」：用什么框架、怎么编排多个组件、怎么观测系统运行状态。


## 5.1 前置知识

第 1–4 讲全部内容。


## 5.2 核心概念：Agent 工程的三大支柱

CS329Z 把 Agent 工程拆成三大支柱：**数据（Data）**——Agent 需要什么样的输入和训练数据；**系统（System）**——Agent 的架构设计和组件编排；**评测（Evals）**——怎么度量系统好不好。这三大支柱贯穿全课。


### DSPy：用编程替代手写 Prompt

DSPy 是 Stanford 开发的框架，核心思想是**把 Prompt 工程变成软件工程**。传统方式：你手写 prompt，反复调试措辞。DSPy 方式：你声明「输入是什么、输出是什么」（Signature），框架自动优化 prompt 措辞和 few-shot 示例。

```python
# DSPy 最小示例
import dspy

class QA(dspy.Signature):
    """根据上下文回答问题"""
    context = dspy.InputField()
    question = dspy.InputField()
    answer = dspy.OutputField()

# 定义模块
qa = dspy.ChainOfThought(QA)
# 用优化器自动调 prompt
from dspy.teleprompt import BootstrapFewShot
optimizer = BootstrapFewShot(metric=my_metric)
optimized_qa = optimizer.compile(qa, trainset=my_trainset)
```

DSPy 的三个核心概念：**Signature**（声明输入输出格式）、**Module**（可调用的处理单元，如 ChainOfThought）、**Optimizer**（自动搜索最优 prompt 的算法，如 BootstrapFewShot、MIPROv2）。


### LangGraph：用图编排 Agent 工作流

LangGraph 把 Agent 工作流建模为**有向图**：节点（Node）是处理步骤（调用模型、执行工具、条件判断），边（Edge）是步骤之间的转移规则，状态（State）是在图中传递的数据。

```python
# LangGraph 最小示例
from langgraph.graph import StateGraph, END

# 定义状态
class AgentState(TypedDict):
    messages: list
    next_step: str

# 定义节点
def call_model(state):
    response = llm.invoke(state["messages"])
    return {"messages": state["messages"] + [response]}

def call_tool(state):
    result = execute_tool(state["messages"][-1])
    return {"messages": state["messages"] + [result]}

# 构图
g = StateGraph(AgentState)
g.add_node("model", call_model)
g.add_node("tool", call_tool)
g.set_entry_point("model")
g.add_conditional_edges("model", should_continue,
    {"tool": "tool", "end": END})
g.add_edge("tool", "model")  # 工具结果回到模型
app = g.compile()
```


## 5.3 课件要点：开源 Agent Harness 巡礼

课件对比了三个开源 Agent Harness 的设计哲学：

| Harness | 特点 | 适用场景 |
| --- | --- | --- |
| mini-SWE-agent | 极简——核心循环不到 100 行 | 学习 Agent 原理、快速原型 |
| OpenHands | 全功能——沙箱、多工具、可视化 | 生产级 Coding Agent |
| Pi | 轻量——专注上下文效率 | 资源受限场景 |


## 5.4 课件要点：可观测性

Agent 系统的调试比普通软件困难得多——失败往往不是程序崩溃，而是「模型做出了错误的决策」。可观测性工具（如 LangSmith、Langfuse）记录每一步的输入输出、工具调用、延迟和成本，让你能回放完整的执行轨迹（Trajectory）找到问题根源。


## 5.5 指定阅读

- Khattab et al.《DSPy》(ICLR 2024)——DSPy 原始论文
- LangGraph 官方文档——图编排概念
- Arena.ai《HarnessTax》(2026)——Harness 对 Agent 性能的影响分析


## 5.6 动手实验

**实验：用 LangGraph 重构你的工具调用循环**。把第 4 讲手写的工具调用循环改造成 LangGraph 图：model 节点调用 LLM，tool 节点执行工具，条件边决定「继续调工具还是结束」。对比手写版和框架版的代码量和可读性。


# 第 6 讲　Agent 设计模式（Agent Design Patterns & Scaffolds）


## 6.0 本讲定位

这一讲是「设计模式」课——就像软件工程有「单例模式」「观察者模式」，Agent 工程也有一套成熟的设计模式。学会这些模式，你就能在面对新任务时快速判断「该用哪种架构」。


## 6.1 前置知识

第 1 讲的 Workflow vs Agent 区分。


## 6.2 核心概念：五种工作流模式

课件基于 Anthropic 的《Building Effective AI Agents》一文，总结了五种可组合的工作流模式：


### 模式一：Prompt Chaining（提示链）

把任务拆成固定的多步，每步调用一次 LLM，前一步的输出是后一步的输入。比如「写论文」拆成「列大纲→写初稿→润色」。适用：任务可以清晰分解为顺序步骤。


### 模式二：Routing（路由）

先对输入分类，然后路由到不同的处理流程。比如客服系统：技术问题→技术知识库，账单问题→账单系统，闲聊→通用对话。适用：输入有多种明确类别。


### 模式三：Parallelization（并行）

同时调用多个 LLM 实例处理同一任务的不同方面，然后汇总。比如「同时检查代码的安全性、性能、可读性」。适用：任务的多个维度可以独立评估。


### 模式四：Orchestrator-Workers（编排者-工作者）

一个中央 LLM 负责分解任务并分配给多个 Worker LLM，每个 Worker 独立完成后返回结果。与并行的区别：编排者动态决定分配什么子任务，而非预先固定。


### 模式五：Evaluator-Optimizer（评估者-优化者）

一个 LLM 生成结果，另一个 LLM 评估并给出反馈，循环直到质量达标。比如「写代码→审查→修改→再审查」。适用：有明确的质量评估标准。


## 6.3 核心概念：推理与行动模式


### ReAct：推理与行动交替

ReAct 是最基础的 Agent 模式：每一步先生成一段推理（Thought），再生成一个动作（Action），观察结果后继续。推理帮助模型维护计划，行动帮助模型获取新信息。


### Reflexion：从失败中学习

Reflexion 在 ReAct 基础上加了「反思」：任务失败后，模型生成一段自我批评（哪里做错了、下次怎么改进），这段反思存入记忆，下次尝试时参考。相当于 Agent 的「错题本」。


### CodeAct：用代码作为动作

CodeAct 让模型直接生成可执行代码作为动作（而不是结构化的 JSON 工具调用）。优势：代码的表达能力远超 JSON——可以写循环、条件、变量赋值。劣势：需要沙箱保证安全。


### ReWOO：推理与工作分离

ReWOO 把「规划」和「执行」分离：Planner 一次性生成完整计划（不调用工具），Worker 按计划批量执行工具，Solver 汇总结果生成答案。优势：减少 LLM 调用次数，降低成本。


## 6.4 核心概念：上下文管理模式

长任务的上下文会不断膨胀，课件介绍了四种管理模式：

- **事件日志（Event Log）**：保留完整的操作记录，但定期裁剪旧事件。
- **压缩（Compaction）**：定期把历史消息摘要成更短的形式，保留关键信息。
- **观察掩码（Observation Masking）**：隐藏不再相关的工具返回结果，减少干扰。
- **上下文隔离（Context Isolation）**：不同子任务使用独立的上下文，避免交叉污染。


## 6.5 指定阅读

- Yao et al.《ReAct》(ICLR 2023)——ReAct 原始论文
- Schluntz & Zhang《Building Effective AI Agents》(Anthropic, 2024)——设计模式权威指南
- Shinn et al.《Reflexion》(NeurIPS 2023)——反思机制
- Wang et al.《CodeAct》(ICML 2024)——代码即动作


## 6.6 动手实验

**实验：实现 ReAct 和 Reflexion 的对比**。步骤：①用 ReAct 模式写一个简单 Agent（推理→行动→观察循环）；②给它一个它第一次会做错的任务；③失败后，加入 Reflexion 机制——让模型生成自我批评，存入记忆，重试；④对比两次尝试的成功率。


# 第 7 讲　Agent 记忆架构（Agent Memory Architectures）


## 7.0 本讲定位

人类有短期记忆和长期记忆——你能记住刚才说过的话（短期），也能记住多年前的事（长期）。Agent 也需要这两种记忆：**记住当前对话的上下文**（工作记忆）和**记住跨会话的用户偏好**（长期记忆）。这一讲讲怎么设计和实现 Agent 的记忆系统。


## 7.1 前置知识

第 2 讲的上下文窗口，第 3 讲的 RAG。


## 7.2 核心概念：四种记忆类型

CS329Z 借鉴认知科学，把 Agent 记忆分为四类：

| 记忆类型 | 存什么 | 类比 | 技术实现 |
| --- | --- | --- | --- |
| 工作记忆（Working） | 当前对话的上下文 | 你正在心算的数字 | 消息历史（Message History） |
| 情景记忆（Episodic） | 过去具体事件的记录 | 你记得上次去某餐厅吃了什么 | 对话日志 + 检索 |
| 语义记忆（Semantic） | 抽象出的事实和偏好 | 你知道自己喜欢意大利菜 | 结构化知识库 / 向量数据库 |
| 程序记忆（Procedural） | 怎么做某事的技能 | 你会骑自行车 | Skill 库 / Few-shot 示例 |


## 7.3 核心概念：四种记忆架构模式


### 模式一：技能库（Skill Library）

把 Agent 学会的技能（成功的操作流程）存成可复用的模板。下次遇到类似任务时，检索最相关的技能模板作为参考。类比：厨师记住菜谱，遇到类似菜品时参考。


### 模式二：分层记忆（MemGPT 风格）

MemGPT 把上下文窗口比作「内存」，把外部存储比作「磁盘」。Agent 自己决定什么时候把信息从「内存」换出到「磁盘」，什么时候从「磁盘」读回「内存」。这让 Agent 在有限的上下文窗口内处理无限长的对话。


### 模式三：反思式记忆巩固

定期（比如每次对话结束后）让模型回顾对话内容，提炼出「值得长期记住的信息」写入长期记忆。不是简单保存全部对话（太多噪音），而是提炼精华。


### 模式四：记忆维护

长期记忆需要定期维护：合并重复条目、删除过时信息、解决矛盾。否则记忆库会越积越多，检索质量下降。


## 7.4 课件要点：记忆的开放问题

- **遗忘**：什么时候该忘记？旧信息可能过时甚至有害。
- **隐私**：记忆库存了用户的个人信息，怎么保护？
- **过度个性化**：记忆太多导致 Agent 对新用户也套用老用户的偏好。


## 7.5 指定阅读

- Sumers et al.《Cognitive Architectures for Language Agents》(TMLR 2024)——认知架构综述
- Packer et al.《MemGPT》(2023)——分层记忆管理
- Park et al.《Generative Agents》(UIST 2023)——生成式 Agent 的记忆系统


## 7.6 动手实验

**实验：给你的 Agent 加上语义记忆**。步骤：①创建一个 memories.json 文件；②每次对话结束后，用 LLM 提炼「用户偏好/重要事实」存入文件；③下次对话开始时，检索相关记忆注入 system prompt；④测试：第一轮对话告诉 Agent 你喜欢简短的回答，第二轮新对话观察 Agent 是否记得。


# 第 8 讲　多智能体系统（Multi-Agent Systems）


## 8.0 本讲定位

一个 Agent 能做的事有限，多个 Agent 协作能做更复杂的事。但多智能体系统不是简单地「多开几个实例」——它们需要通信、协调、分工。这一讲讲什么时候该用多智能体、怎么设计协作架构、以及多智能体系统为什么会失败。


## 8.1 前置知识

第 1 讲的 Agent 概念，第 6 讲的设计模式。


## 8.2 核心概念：单 Agent vs 多 Agent

**什么时候用单 Agent**：任务流程相对线性、上下文可以共享、不需要并行处理。大多数任务用单 Agent 就够了。**什么时候考虑多 Agent**：任务可以明确分解为独立子任务、需要不同专业角色、需要并行处理提高效率。

> ⚠️ **注意**：⚠ 注意：多智能体系统不是银弹。课件引用研究指出，多智能体系统的失败率远高于单 Agent——通信开销、协调失败、错误级联都是常见问题。**默认用单 Agent，只在明确受益时才上多 Agent**。


## 8.3 核心概念：三种编排模式


### Orchestrator（编排者模式）

一个中央 Agent 负责分解任务、分配子任务、汇总结果。其他 Agent 只负责执行分配到的子任务。类比：项目经理 + 团队成员。


### Swarm（蜂群模式）

没有中央协调者，每个 Agent 自主决策，通过共享信息间接协调。类比：蚁群——没有蚁后指挥，但群体能完成复杂任务。


### Hybrid（混合模式）

结合两者：有中央协调者处理大方向，但子任务内部各 Agent 有一定自主权。这是实践中最常用的模式。


## 8.4 核心概念：通信机制

- **Handoffs（交接）**：一个 Agent 完成任务后，把上下文和控制权交给下一个 Agent。
- **Message Boards（消息板）**：Agent 之间通过共享消息区域异步通信。
- **A2A 协议**：Google 提出的 Agent-to-Agent 标准协议，定义了 Agent 之间的发现和通信格式。


## 8.5 课件要点：多智能体为什么失败

课件引用 NeurIPS 2025 论文《Why Do Multi-Agent LLM Systems Fail?》，总结了三大失败原因：

- **通信带宽不足**：该传的信息没传到——Agent A 发现了关键线索，但没有告诉 Agent B。
- **共享状态缺失**：各 Agent 各自为战，不知道其他 Agent 在做什么，导致重复劳动或互相矛盾。
- **错误级联**：一个 Agent 的错误输出成为另一个 Agent 的输入，错误被放大传播。


## 8.6 指定阅读

- Wu et al.《AutoGen》(COLM 2024)——多 Agent 对话框架
- Cemri et al.《Why Do Multi-Agent LLM Systems Fail?》(NeurIPS 2025)——失败分析
- Kim et al.《Towards a Science of Scaling Agent Systems》(2025)——规模化理论


## 8.7 动手实验

**实验：写一个双 Agent 协作系统**。步骤：①创建「研究员」Agent（负责搜索和整理信息）和「作家」Agent（负责基于信息写文章）；②用 Orchestrator 模式协调——编排者接收用户请求，先分配给研究员，收到结果后分配给作家；③记录完整的通信日志；④对比单 Agent 完成同样任务的质量。
