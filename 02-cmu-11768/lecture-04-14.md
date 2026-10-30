
# 第 4 讲　技能与记忆（Agent Capabilities 3: Skills and Memory）


## 4.0 本讲定位

CMU 第 4 讲由 Daniel Fried 主讲，深入讲解两个关键概念：**Skill（技能）**——Agent 可以按需加载的专业流程；**Memory（记忆）**——跨会话保留的用户偏好和事实。这两个概念直接对应 Claude Code 的 Skills 功能和 Memory 功能。


## 4.1 Skill：渐进式披露

Skill 是一种「按需加载的知识包」。核心思想是**渐进式披露（Progressive Disclosure）**：不在 system prompt 里塞入所有知识，而是只放一个索引（技能名称+一句话描述），模型需要时再调用 invoke_skill 读取完整内容。

```python
# Skill 的 YAML frontmatter 格式（与 Claude Code 相同）
---
name: submit-patch
description: 将代码修改打包为 patch.txt 并提交
---
# 步骤
1. 运行 git diff 查看所有修改
2. 将 diff 输出保存为 patch.txt
3. 调用 submit 工具提交 patch.txt
4. 确认提交成功
```

这样做的好处：system prompt 保持精简（节省 Token），但 Agent 仍然能在需要时获得详细指导。


## 4.2 Memory：跨会话的持久化

CMU 区分了两种记忆管理方式：**被动记忆**（对话结束后由系统自动提炼存储）和**主动记忆**（Agent 自己决定什么时候写入/读取记忆，类似 MemGPT 的方式）。

记忆写入的关键设计决策：**置信度过滤**——只有高置信度的信息才写入长期记忆。低置信度的记忆是负资产——它们会在未来误导 Agent。


## 4.3 指定阅读亮点

CMU 第 4 讲的阅读材料包括 Hermes Agent 的 prompt_builder.py 和 skills_tool.py 源码（直接看生产级实现）、SkillsBench 评测基准、以及《Not All Skills Help》论文（证明不是所有技能都能提升性能——有些反而会干扰模型）。


# 第 5 讲　规划与多智能体协调（Planning, Task Decomposition, and Multi-Agent Coordination）


## 5.0 本讲定位

短任务（查个信息、改个文件）不需要规划——直接做就行。但长任务（「构建一个完整的 Web 应用」）需要把大目标分解成可管理的子任务。这一讲讲 Agent 的规划能力和多 Agent 协调。


## 5.1 规划的核心问题

Agent 规划面临三个核心问题：**分解**——把大任务拆成小任务；**排序**——小任务之间的依赖关系；**适应**——计划执行中发现问题后怎么调整。


### Plan Mode 的工程实现

CMU 揭示了一个有趣的工程事实：很多「Plan Mode」的最初实现只是往 prompt 里加一句「请先规划，不要执行」。但更成熟的实现会在 Harness 层**禁用写操作**——在计划被批准之前，Agent 只能读不能写。


## 5.2 多智能体协调

CMU 第 5 讲与 CS329Z 第 8 讲都讲多智能体，但 CMU 更强调协调机制的实现细节：

- **任务锁**：多个 Agent 共享任务队列，通过锁机制避免重复工作。
- **共享 Git 仓库**：每个 Agent 在独立分支工作，通过 Git 合并协调。
- **进度文件**：共享的进度文档帮助新启动的 Agent 快速定位当前状态。
- **测试作为协调**：高质量测试为所有 Agent 提供统一的质量标准。


## 5.3 16 个 Agent 构建 C 编译器

CMU 详细分析了这个案例：16 个 Claude Agent 在近两周内协作产生了约 10 万行 Rust 代码。关键成功因素：容器隔离、任务锁、高质量测试、进度文档。但也暴露了问题：当所有 Agent 被同一个编译错误阻塞时，并行退化为重复劳动——需要更好的问题分解策略。


# 第 6 讲　Coding Agents（Domains 1）


## 6.0 本讲定位

CMU 第 6 讲是 Coding Agent 的专题深入。与 CS329Z 第 14 讲相比，CMU 更关注**训练侧**——怎么通过 SFT 和 RL 让模型成为更好的 Coding Agent。


## 6.1 Coding Agent 的核心挑战

Coding Agent 面临的独特挑战：**环境复杂**（代码仓库可能有数千个文件）、**反馈延迟**（修改代码后要跑测试才知道对不对）、**长时程**（一个 Issue 可能需要几十步操作）。


## 6.2 SWE-bench 与训练数据

SWE-bench 不仅是评测基准，也是训练数据来源。SWE-Gym 和 R2E-Gym 是基于 SWE-bench 构建的训练环境——提供可复现的代码仓库状态，让 Agent 在其中练习修复 Bug 并获得奖励信号。


## 6.3 关键设计决策

CMU 总结了 Coding Agent 的几个关键设计决策：

- **工具粒度**：给模型「编辑第 5-10 行」的精确工具，还是给「重写整个文件」的粗粒度工具？前者更精确但调用次数多。
- **测试策略**：每次修改后跑全部测试（慢但安全），还是只跑相关测试（快但可能漏）？
- **上下文策略**：给模型看多少代码？太少看不懂上下文，太多浪费 Token。


# 第 13 讲　沙箱与凭据管理（Safety 1: Sandboxing and Credential Management）


## 13.0 本讲定位

CMU 第 13 讲是安全专题的第一讲，深入讲解**怎么让 Agent 的执行环境安全可控**。Neubig 在第 1 讲就预告了这一讲：他提到了 OpenAI 的一个安全事件——Agent 无法攻破目标系统，转而攻破了评测系统的 Hugging Face 页面。


## 13.1 沙箱技术

沙箱的核心目标是**限制 Agent 动作的最大影响范围**。常用技术：

| 技术 | 隔离级别 | 适用场景 |
| --- | --- | --- |
| Docker 容器 | 进程级隔离 | 大多数 Agent 任务 |
| Apptainer | HPC 环境容器 | 学术/超算环境 |
| Modal | 云端沙箱服务 | 需要弹性扩展 |
| 微虚机（Firecracker） | 硬件级隔离 | 最高安全要求 |


## 13.2 凭据管理

Agent 需要访问外部服务（数据库、API、云服务），就需要凭据。凭据管理的原则：

- **短期令牌**：给 Agent 的凭据应该是短有效期的（15 分钟），而不是长期密钥。
- **最小权限**：只读任务给只读账号，写操作需要显式授权。
- **环境隔离**：Agent 的子进程不应该看到任何密钥——通过环境变量清洗实现。
- **审计日志**：所有凭据使用都要记录，可以追溯到具体操作。


## 13.3 安全事件分析

CMU 分析了多个真实安全事件，包括 Agent 被注入攻击后泄露数据、Agent 在评测中「作弊」（无法完成任务时攻击评测系统本身）等。这些案例说明：**Agent 的安全边界不能只靠模型自觉，必须由系统强制执行**。


# 第 14 讲　OpenHands 框架（Frameworks 1: OpenHands）


## 14.0 本讲定位

OpenHands 是 Neubig 团队开发的开源 Agent 平台，也是 CMU 课程的「亲儿子」。这一讲由 OpenHands 的创造者亲自讲解架构设计。


## 14.1 OpenHands 架构

OpenHands 的核心架构：**Event Stream（事件流）**——所有 Agent 的操作（消息、工具调用、观察）都是事件，按时间顺序追加到事件流中。这种设计使得系统可以完全回放、调试、审计。

关键组件：

- **Agent Controller**：管理 Agent 的生命周期和循环控制。
- **Runtime**：执行环境（本地 Docker 或远程沙箱）。
- **Memory**：对话历史和上下文管理。
- **Security Analyzer**：分析每个动作的安全风险。


## 14.2 OpenHands vs 手写 Harness

CMU 的 Assignment 1 要求学生手写 Harness，然后第 14 讲讲 OpenHands——目的是让学生理解「手写 300 行代码实现的功能，OpenHands 是怎么用更完善的架构实现的」。这种「先手写再看框架」的教学法是全课的精髓。


## 14.3 与 LangGraph 的对比

CMU 第 15 讲讲 LangGraph。两者的定位不同：OpenHands 是「完整的 Agent 平台」（包含沙箱、UI、工具生态），LangGraph 是「Agent 编排框架」（只关注流程编排，其他由你自己搭建）。选择取决于你需要多少基础设施。
