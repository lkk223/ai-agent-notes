
# 卷二导读

CMU 11-768 与 CS329Z 的最大区别是**技术深度**：CS329Z 讲「是什么、怎么用」，CMU 讲「底层怎么实现、为什么这样设计」。CMU 的讲师 Graham Neubig 是 OpenHands 的创造者，Daniel Fried 是 Agent 领域的知名研究者。

本卷覆盖用户指定的讲次：**第 1–5 讲**（Agent 能力基础：概念、工具、上下文、记忆、规划）、**第 6 讲**（Coding Agents）、**第 13 讲**（安全）、**第 14 讲**（OpenHands 框架）。CMU 的课程录像在 YouTube 公开，建议配合观看。


# 第 1 讲　什么是 Agent？（Course Overview）


## 1.0 本讲定位

CMU 第一讲与 CS329Z 第一讲覆盖相似主题，但角度不同：CS329Z 从「系统设计」角度切入，CMU 从「经典 AI 定义 → 现代 LLM Agent 的映射」切入，更强调形式化定义和系统分层。


## 1.1 经典 Agent 定义

课件引用 Russell & Norvig 的经典定义：**Agent 是任何能通过传感器感知环境、并通过执行器作用于环境的事物**。这个定义涵盖了从恒温器到机器人的一切自主系统。

经典 Agent 的四个核心概念：

| 概念 | 含义 | 例子 |
| --- | --- | --- |
| Environment（环境） | Agent 所处的外部世界 | 代码仓库、网页、操作系统 |
| State / Observation（状态/观察） | 环境的真实情况 / Agent 能看到的信息 | 完整文件系统 / Agent 读到的某个文件内容 |
| Action（动作） | Agent 对环境施加的操作 | 编辑文件、执行命令、点击按钮 |
| Reward（奖励） | 对结果质量的数值反馈 | 测试通过=1，失败=0 |

> 💡 **讲师批注**：讲师批注：State 和 Observation 的区分非常重要。环境的真实状态可能极其复杂（整个文件系统+进程+网络状态），但 Agent 每次只能看到一小部分（Observation）。Agent 的决策质量受限于它能观察到什么。


## 1.2 从 LLM 到 Agent：Action 即 Token

CMU 第一讲最精彩的部分是解释「LLM 怎么从文字生成器变成行动者」。核心洞察：**模型从不直接执行动作——它生成表示动作的 Token，Harness 解析并执行这些 Token**。

完整链路：模型生成 Reasoning Token（思考过程）→ 生成 Tool Call Token（结构化的调用请求）→ Harness 解析、校验、执行 → 环境返回 Tool Result → 结果被编码为 Observation Token 送回模型。

**跨越模型与环境边界的是 Harness，不是模型本身**。模型始终活在 Token 的世界里。


### Tool Definition / Tool Call / Tool Result

```python
# Tool Definition（工具定义——告诉模型有什么可用）
{"name": "read_file",
 "description": "读取 UTF-8 文件内容",
 "parameters": {"path": {"type": "string"}}}

# Tool Call（工具调用——模型生成的动作请求）
{"id": "call_7", "name": "read_file",
 "arguments": {"path": "test.py"}}

# Tool Result（工具结果——环境返回的观察）
{"role": "tool", "tool_call_id": "call_7",
 "name": "read_file", "content": "def add(a,b): ..."}
```


## 1.3 Agent 系统分层

CMU 提出了 Agent 系统的六层架构，这是理解 Agent 工程全貌的关键框架：

| 层级 | 组件 | 职责 |
| --- | --- | --- |
| 模型层 | LLM | 根据 Context 生成 Reasoning 和 Action Token |
| Harness 层 | Agent 框架 | Prompt 组装、工具执行、状态管理、循环控制 |
| 沙箱层 | Sandbox | 隔离执行环境，限制资源、网络、文件、凭据 |
| 推理服务层 | LM Inference | 模型服务、批处理、KV Cache、流式输出 |
| 训练层 | Training System | 数据准备、Rollout、奖励计算、模型更新 |
| 观测层 | Observability | Trace 记录、质量监控、延迟/成本分析 |


## 1.4 两条改进路线：Training vs Harness

CMU 第一讲的重要论点：Agent 能力的提升有两条路线——**训练模型**（改变模型参数）和**工程化 Harness**（改变模型周围的系统）。

经验判断：新问题往往先在 Harness 层被解决（快速迭代）；当问题足够普遍后，一部分能力会被训练进模型（长期固化）。但 Harness 永远不会完全消失——权限控制、业务规则、用户偏好始终需要系统层处理。


## 1.5 Agent 需要的六类能力

| 能力 | 要解决的问题 | 典型失败 |
| --- | --- | --- |
| 准确工具调用 | 选对工具、生成合法参数 | 工具名错误、JSON 破损 |
| 长上下文连贯 | 在长轨迹中保留约束 | 忘记用户要求、重复工作 |
| 可定制性 | 适应用户/组织的偏好 | 无视代码规范 |
| 复杂任务管理 | 分解、规划、完成长任务 | 做到一半迷失目标 |
| 环境理解 | 正确解释环境数据 | 看错网页、误解日志 |
| 安全 | 在权限边界内完成 | 泄露凭据、越权操作 |


## 1.6 与 Assignment 1 的关系

CMU 的 Assignment 1（Build an Agent Harness）要求学生实现一个完整的 ReAct Agent Loop，在沙箱中修复一个 Chess App 的 Bug。作业把第一讲的所有概念落地：Context 构造、Tool 定义与执行、错误恢复、上下文压缩、Skill 渐进式披露。


# 第 2 讲　工具使用（Agent Capabilities 1: Tool Use）


## 2.0 本讲定位

CS329Z 第 4 讲也讲工具调用，但 CMU 第 2 讲深入到**模型内部怎么编码工具调用**——不同模型（Qwen、DeepSeek、Mistral）的 Chat Template 是怎么把工具定义和工具调用渲染成 Token 的。理解这个层面，你就明白了一半的「模型调错工具」的 Bug。


## 2.1 Chat Template：工具调用的底层编码

每个模型都有自己的 Chat Template——一种把结构化消息（system/user/assistant/tool）渲染成纯文本 Token 序列的模板。工具定义、工具调用、工具结果在 Token 层面的表示方式因模型而异。

例如，Qwen 使用特殊的 <tool_call> 标签包裹工具调用，DeepSeek 使用不同的编码方式。当你切换模型时，如果 Harness 没有正确适配新模型的 Chat Template，工具调用就会失败。


## 2.2 工具调用的约束解码

CMU 深入讲解了约束解码（Constrained Decoding）的实现原理：在模型生成每个 Token 时，根据 JSON Schema 的语法规则，把不合法的 Token 的概率设为零。这保证了模型输出的 JSON 一定是合法的——但**合法不等于正确**，模型仍然可能生成合法但语义错误的调用。


## 2.3 工具设计原则

CMU 引用 Anthropic 的工具设计指南，总结了五条原则：

- **精选而非堆砌**：给模型 5 个好工具，胜过给 50 个平庸工具。工具太多会导致「上下文混淆」。
- **命名空间分组**：相关工具用统一前缀（如 file_read、file_write、file_search），帮助模型理解工具之间的关系。
- **返回语义化数据**：工具返回的内容应该对模型友好——不要返回原始二进制或冗长日志。
- **优化 Token 效率**：工具返回的内容会占用上下文窗口——精简返回，只给模型需要的信息。
- **工具描述是性能杠杆**：描述写得好，工具调用的准确率能提升 20-30%。


## 2.4 指定阅读亮点

CMU 第 2 讲列出了 26 篇阅读材料，包括 MCP 最新规范、Qwen/DeepSeek 的工具调用模板源码、OpenHands 的 ToolDefinition 实现。这些一手资料比任何教科书都更有价值。


# 第 3 讲　长上下文管理（Context Management for Long-Context Agents）


## 3.0 本讲定位

Agent 每执行一步，历史就多一段。跑 50 步后，上下文可能有 10 万 Token。这一讲讲**上下文膨胀的代价和解决方案**。CMU 这一讲的阅读材料有 44 篇，是全课最重的。


## 3.1 上下文膨胀的三重代价

- **成本**：每次 API 调用都要发送完整上下文，Token 费用随历史长度线性增长。
- **延迟**：Prefill（处理输入）时间随上下文长度增长，首 Token 延迟变高。
- **质量**：模型对长上下文中部信息的利用率下降（Lost in the Middle），且超过 100K Token 后模型倾向于重复过去行为（Context Distraction）。


## 3.2 KV Cache 与缓存定价

CMU 详细讲解了 KV Cache 的经济学：Agent 的多轮调用中，大量前缀是重复的（system prompt + 早期历史）。KV Cache 允许模型复用之前计算过的前缀的 Key-Value 矩阵，避免重复计算。

主流厂商（OpenAI、Anthropic、DeepSeek、Kimi 等）都提供缓存定价——缓存命中的 Token 价格远低于新 Token。这意味着**保持前缀稳定**（不要频繁修改 system prompt 的开头部分）可以大幅降低成本。


## 3.3 上下文压缩策略


### 摘要压缩（Compaction）

定期用 LLM 把历史消息摘要成更短的形式。关键设计：**不可压缩层**——某些消息（原始任务描述、安全约束、用户的明确禁令）永远不参与压缩。


### 观察掩码（Observation Masking）

隐藏不再相关的工具返回结果。比如 Agent 在第 3 步读取了文件 A，在第 20 步已经不需要文件 A 的内容了，就把那次工具返回标记为「已折叠」，只保留摘要。


### 子 Agent 隔离

把独立的子任务分配给子 Agent，子 Agent 有独立的上下文。子 Agent 完成后只返回摘要结果，不把完整过程带回主 Agent 的上下文。


## 3.4 生产级 Agent 的上下文策略

CMU 分析了 Codex、OpenCode、Pi、Hermes Agent、OpenHands 五个生产级 Agent 的上下文管理策略，发现它们都采用了「分层上下文」：

- **固定层**：System prompt + 工具定义（不变，利用 KV Cache）。
- **半固定层**：项目上下文 / CLAUDE.md（偶尔更新）。
- **动态层**：对话历史 + 工具结果（持续增长，定期压缩）。
- **按需层**：Skill 内容、检索结果（需要时注入，用完移除）。
