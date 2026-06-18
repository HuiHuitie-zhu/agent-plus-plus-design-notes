# agent+++ Design Notes

> 中文版  
> 状态：公开分享版草稿  
> 日期：2026-06-18

agent+++ 是一个面向长期项目的人机协作 Agent 工作台设计。

它不是普通聊天机器人，也不是只会执行命令的代码 Agent。agent+++ 关注的是一个更难的问题：

> AI 如何持续推进一个真实项目，而不是只完成一次孤立任务？

这组设计笔记整理了 agent+++ 当前最核心的架构判断：Thinker、Reflex Layer、Feedback Bus、Observation Contract、Global Precipitation Memory 和 Project Memory 如何协作，把人的碎片想法推进成可执行、可验证、可沉淀的项目资产。

---

## 一句话定位

**agent+++ 是一个本地优先的项目型 Agent 操作系统：用深度 Thinker 做决策拆解，用 Flash 执行层做并行推进，用 Observation Contract 保证反馈不失真，用全局沉淀层和项目记忆让长期项目可以接着做。**

普通 Agent 更像：

```text
用户提出任务 -> Agent 执行 -> 返回结果
```

agent+++ 想做的是：

```text
模糊想法
  -> 澄清
  -> 决策拆解
  -> 多线程执行
  -> 结构化观察
  -> 阶段收束
  -> 记忆沉淀
  -> 新阶段继续
```

---

## 当前公开资料

建议阅读顺序：

1. [agent-plus-plus-value-points.md](agent-plus-plus-value-points.md)  
   项目总览：四大核心模块、价值判断和产品定位。
   英文版：[agent-plus-plus-value-points.en.md](agent-plus-plus-value-points.en.md)

2. [observation-contract.md](observation-contract.md)  
   Observation Contract：把工具输出变成可推理、可审计、可沉淀的观察资产。
   英文版：[observation-contract.en.md](observation-contract.en.md)

3. [global-precipitation-memory.md](global-precipitation-memory.md)  
   Global Precipitation Memory：全局沉淀层、项目记忆投影、保留期、幻觉期和遗忘机制。
   英文版：[global-precipitation-memory.en.md](global-precipitation-memory.en.md)

4. [thinker-architecture.md](thinker-architecture.md)  
   Thinker Architecture：项目决策拆解大师、多线程调频、收束站和代际切换。
   英文版：[thinker-architecture.en.md](thinker-architecture.en.md)

5. [why-deepseek.md](why-deepseek.md)  
   为什么选择 DeepSeek：模型底座、Flash / Pro 分工和 token 高利用率。
   英文版：[why-deepseek.en.md](why-deepseek.en.md)

---

## 四大核心模块

### 1. Thinker

Thinker 是项目决策拆解大师。

它用 Pro 模型做高价值判断，把复杂项目拆成可并行推进的线程，并在执行过程中持续调频：

```text
祖 Thinker
  -> Thread Plan
  -> Decision Inheritance Packet
  -> Flash 并行执行
  -> Thread Observation
  -> Tuning Decision
  -> Convergence Dock
  -> Generational Handoff
```

Thinker 的目标不是让 Pro 模型从头到尾慢慢跑，而是让 Pro 负责最深决策，让 Flash agents 负责最大吞吐。

### 2. Reflex Layer

Reflex Layer 是快速执行层。

它负责读写文件、调用工具、跑命令、搜索、执行小步任务。它不做高层方向判断，只根据 Thinker 给出的线程计划和继承包推进任务。

### 3. Feedback Bus / Observation Contract

Feedback Bus 不只是日志压缩器。

它通过 Observation Contract 把工具输出拆成多个维度：

```text
provenance
determinism
epistemic
semantic_role
actionability
fidelity
```

这让系统知道：

```text
什么是事实？
什么只是模型解释？
什么会阻塞下一步？
什么需要保留原文证据？
什么可以进入记忆？
```

### 4. Project Memory / Global Precipitation Memory

Project Memory 不是第一沉淀层。

第一沉淀层应该是全局的：

```text
Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

全局沉淀层负责暂存尚未归属的观察、Pro 思考记录和用户认知；Project Memory 是从全局沉淀层按项目语境析出的局部视图。

---

## 为什么选择 DeepSeek 作为模型底座

agent+++ 的模型策略不是“只用一个最强模型”，而是：

```text
DeepSeek Flash 承担高频执行和反馈处理
DeepSeek Pro 承担深度思考和项目决策
```

选择 DeepSeek 做底座，主要有四个原因。

### 1. 成本结构适合多线程 Agent

agent+++ 的核心不是单次问答，而是长期项目中的大量小步执行：

```text
线程拆解
工具调用
反馈解析
阶段收束
记忆沉淀
局部升级
```

这些动作会产生大量模型调用。如果每一步都用昂贵大模型，多线程系统很快会失去性价比。

DeepSeek 的 Flash / Pro 分层适合 agent+++ 的结构：

```text
高频、低风险、可验证动作 -> Flash
低频、高价值、方向判断 -> Pro
```

### 2. 推理能力适合 Thinker

Thinker 需要的不是普通聊天能力，而是：

- 拆解复杂目标
- 保留约束
- 识别风险
- 生成决策链
- 处理反馈后重新调频
- 做阶段收束和代际切换

DeepSeek Pro 适合承担这种高价值判断节点。

### 3. Flash 模型适合 Reflex Layer

Reflex Layer 需要的是速度、低成本和大量重复执行：

- 读文件
- 总结局部输出
- 解析工具结果
- 执行线程中的小任务
- 生成结构化回传

这些任务不需要每次都调用最强模型。Flash 模型配合 Observation Contract，可以把大量执行反馈整理成 Thinker 可用的高质量观察。

### 4. 适合本地优先工作台的工程节奏

agent+++ 的目标是本地优先、可控、可审计。

DeepSeek 的模型层可以被封装成可替换的 LLM backend：

```text
Thinker Model
Reflex Model
Observation Model
Memory Extraction Model
```

这意味着 DeepSeek 是当前默认底座，但系统设计不被单一模型绑定。

---

## DeepSeek 在 agent+++ 中支持到什么程度

agent+++ 对 DeepSeek 的支持可以分成四层。

### 1. 基础对话与流式输出

用于普通对话、项目澄清、阶段总结和用户沟通。

### 2. 结构化决策输出

Thinker 需要输出结构化对象：

```text
ThreadPlan
DecisionInheritancePacket
TuningDecision
ConvergenceStrategy
GenerationalHandoff
ThoughtRecord
```

即使底层模型不是严格函数调用模式，agent+++ 也可以通过结构化提示词、JSON schema 校验和修复循环获得稳定输出。

### 3. 工具调用编排

agent+++ 不依赖模型直接自由调用所有工具。

更稳的方式是：

```text
模型输出执行意图
系统做权限与安全判断
Reflex Layer 调用工具
Observation Contract 回传结果
Thinker 再判断下一步
```

这让 DeepSeek 能参与完整 agent loop，同时避免模型直接越权操作。

### 4. 多模型分工

DeepSeek Flash 和 Pro 在系统中承担不同角色：

```text
Flash:
  - 工具结果整理
  - 局部线程执行
  - 低风险总结
  - 记忆候选初筛

Pro:
  - 祖 Thinker 决策
  - 次级 Thinker 局部难题
  - 阶段收束
  - 代际切换
  - 重要架构判断
```

这不是简单“大小模型混用”，而是把模型能力放进明确的系统角色里。

---

## Token 高利用率设计

agent+++ 不把 token 省下来当目标，而是把 token 用在最值钱的位置。

核心策略是：

### 1. 原始日志不直接进 Thinker

工具输出先进入 Observation Contract：

```text
raw output
  -> facts / risks / artifacts / evidence_refs
  -> summary_for_thinker
```

Thinker 读的是决策材料，不是现场噪音。

### 2. 子线程继承决策骨架，不继承完整上下文

每个 Flash 线程拿到的是 Decision Inheritance Packet：

```text
父代目标
约束
质量标准
当前线程范围
完成条件
升级触发条件
```

这比把整个项目历史塞给每个子 agent 更省 token，也更不容易跑偏。

### 3. 项目记忆来自相关性析出

Project Memory 不是把所有历史都塞进上下文，而是从全局沉淀层按项目和任务析出：

```text
Global Precipitation Layer
  -> relevant memory projection
  -> Project Memory View
  -> Thinker Context
```

### 4. 阶段收束后做代际切换

长期项目不靠无限压缩上下文续命，而是：

```text
旧祖 Thinker
  -> 阶段收束
  -> Generational Handoff
  -> 新祖 Thinker
```

新祖 Thinker 继承阶段结论和边界，不继承完整历史。

一句话：

```text
压缩是把旧材料塞小；
代际切换是只继承结论和边界。
```

### 5. 高保真内容按需打开

Observation Contract 支持 Fidelity Policy：

```text
drop / summary / extract / excerpt / raw_window / full_raw
```

默认不给 Thinker 喂完整日志；只有用户要求、验证需要或关键证据需要时，才打开高保真窗口。

---

## 当前状态

这些文档是 agent+++ 的公开设计笔记，不代表全部功能已经产品化完成。

当前更准确的状态是：

```text
核心架构：已成型
Observation Contract：已有工程雏形
Thinker：设计已定稿，待工程实现
Global Precipitation Memory：设计已成型，待工程实现
Project Memory：需要从项目记忆升级为全局沉淀投影视图
Reflex Layer：已有基础执行能力，待多线程调度增强
UI：需要展示线程、调频、收束、阶段档案
```

这组文档的价值是解释 agent+++ 的系统方向，而不是包装一个已经完全完成的产品。

---

## 核心观点

普通 Agent 追求：

```text
更会回答
更会调用工具
更会自动执行
```

agent+++ 追求：

```text
更会推进项目
更会组织多线程 AI 劳动力
更会保留证据
更会阶段收束
更会形成长期记忆
```

如果说普通 Agent 是一个执行助手，agent+++ 想成为的是：

> 一个能把碎片想法推进成长期项目资产的 AI 项目操作系统。
