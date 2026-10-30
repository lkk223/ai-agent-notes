# AI Agent 课程笔记全集

> 基于 Stanford CS329Z / CS146S · CMU 11-768 · UC Berkeley CS294-196
> 中文原创笔记 · 从零基础到项目实战

## 这是什么

把四门顶尖大学的 AI Agent 课程的公开课件内容，系统整理为中文笔记。不是简单翻译，而是从零基础出发、层层递进的知识重构。

## 四门课的定位

| 课程 | 定位 | 特点 | 适合谁 |
|---|---|---|---|
| Stanford CS329Z | 主线教材 | 19 讲覆盖全栈：概念→工具→RAG→框架→评测→安全 | 零基础入门 Agent 工程 |
| CMU 11-768 | 技术深入 | Chat Template、KV Cache、沙箱等底层细节；OpenHands 作者亲授 | 想理解「框架在底层做了什么」 |
| Stanford CS146S | 工程实践 | MCP、上下文工程、安全审查、Agent 管理 | 日常用 AI 编码工具的工程师 |
| UC Berkeley CS294-196 | 前沿视野 | ReAct 作者、DSPy 作者、AutoGen 作者等一线研究者 | 想了解领域最前沿方向 |

## 推荐学习路径

- **路径 A（零基础，推荐）**：卷一 CS329Z 按讲次顺序学 → 卷五项目实战 → 卷二 CMU 深入 → 卷三 CS146S → 卷四 Berkeley
- **路径 B（有 LLM 基础）**：卷二 CMU 深入 → 卷一 CS329Z 补充 → 卷五项目实战 → 卷三 + 卷四
- **路径 C（只想快速上手）**：卷五项目实战直接做 → 遇到不懂的概念回查卷一对应讲次

## 目录

### 卷一：Stanford CS329Z — Engineering AI Agents
- [第 1-4 讲：Agent 概念、LLM 原理、RAG、工具调用](01-cs329z/lecture-01-04.md)
- [第 5-8 讲：框架编排、设计模式、记忆架构、多智能体](01-cs329z/lecture-05-08.md)
- [第 9-12 讲：优化、数据、评测基础、LLM-as-Judge](01-cs329z/lecture-09-12.md)
- [第 13-16 讲 + HW1/HW2：安全、Coding Agent、主动 Agent、作业](01-cs329z/lecture-13-hw.md)

### 卷二：CMU 11-768 — AI Agents
- [第 1-3 讲：Agent 定义、工具使用、上下文管理](02-cmu-11768/lecture-01-03.md)
- [第 4-14 讲：技能与记忆、规划、Coding Agent、安全、OpenHands](02-cmu-11768/lecture-04-14.md)

### 卷三：Stanford CS146S — The Modern Software Developer
- [10 周完整笔记](03-cs146s/README.md)

### 卷四：UC Berkeley CS294-196 — Agentic AI
- [精选讲座笔记](04-berkeley-cs294/README.md)

### 卷五：项目实战手册
- [4 个完整项目：手写 Harness / MCP Server / RAG / 评测套件](05-projects/README.md)

## 原始课程链接

- CS329Z：https://cs329z.stanford.edu/
- CS146S：https://themodernsoftware.dev/
- CMU 11-768：https://cmu-agents.com/（YouTube 录像公开）
- Berkeley CS294-196：https://rdi.berkeley.edu/agentic-ai/f25

## License

CC BY-NC-SA 4.0 — 非商业用途自由使用，署名+相同方式共享。
