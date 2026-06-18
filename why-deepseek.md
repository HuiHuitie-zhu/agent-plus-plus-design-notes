# 为什么选择 DeepSeek 作为 agent+++ 的模型底座

> 日期：2026-06-19  
> 状态：公开分享版草稿  
> 关联模块：Thinker / Reflex Layer / Observation Contract / Global Precipitation Memory  
> 用途：说明 agent+++ 为什么默认选择 DeepSeek，以及 DeepSeek 在系统中承担到什么程度。

---

## 一句话结论

**agent+++ 选择 DeepSeek，不是因为要绑定某一个模型供应商，而是因为 DeepSeek 的 Flash / Pro 分层、推理能力、成本结构和长上下文能力，适合支撑“深度 Thinker + 高频 Flash 执行 + 高 token 利用率”的 Agent 操作系统。**

agent+++ 的模型策略不是：

```text
所有事情都用一个最强模型
```

而是：

```text
Pro 做深度项目决策
Flash 做高频执行与反馈处理
Observation Contract 提高 token 使用质量
Generational Handoff 避免长期项目上下文污染
```

---

## 为什么不是只追求最强模型

长期项目型 Agent 和普通聊天产品不一样。

它不是一次问答，而是持续运行：

```text
澄清需求
拆解线程
调用工具
解析反馈
搜索补证
局部升级
阶段收束
生成记忆
新阶段继续
```

这些动作会产生大量模型调用。如果所有步骤都用最贵、最深的模型，多线程系统会很快失去可用性。

agent+++ 的判断是：

> 模型能力必须放进角色系统里，而不是无差别使用。

---

## DeepSeek 在 agent+++ 中的角色分工

### DeepSeek Pro：负责深度判断

DeepSeek Pro 适合承担 Thinker 里的高价值节点：

- 祖 Thinker 项目决策
- 复杂目标拆解
- 父代决策链生成
- 次级 Thinker 局部难题
- 阶段收束
- 代际切换
- 重要架构判断
- 高风险取舍

这些节点调用频率不应该太高，但每次都很重要。

```text
低频
高价值
强推理
影响项目方向
```

### DeepSeek Flash：负责高频执行

DeepSeek Flash 更适合 Reflex Layer 和反馈处理：

- 线程内小步执行
- 工具结果整理
- 局部文件理解
- 低风险总结
- Observation Contract 初筛
- 记忆候选初筛
- UI 友好摘要

这些节点调用频率高、数量多、结果通常可验证。

```text
高频
低成本
低延迟
可验证
```

---

## 为什么 DeepSeek 适合多线程 Agent

agent+++ 的 Thinker 架构依赖多线程协作：

```text
祖 Thinker
  -> Thread Plan
  -> 多个 Flash 线程并行执行
  -> 阻塞线程搜索或升级次级 Thinker
  -> Observation Contract 回传
  -> 祖 Thinker 调频
  -> Convergence Dock 收束
```

这要求模型底座同时满足两件事：

1. **深度节点要想得清楚**
2. **高频节点要跑得起**

DeepSeek 的 Pro / Flash 分层刚好适合这个结构。

如果只有强模型，系统会慢且贵。  
如果只有快模型，系统会缺少深度判断。  
agent+++ 需要的是两者组合：

```text
深度决策用 Pro
高频执行用 Flash
```

---

## DeepSeek 能支持到什么程度

### 1. 基础对话与流式输出

用于：

- 普通沟通
- 项目澄清
- 阶段总结
- 用户可读反馈

### 2. Thinker 结构化输出

Thinker 需要稳定输出结构化对象：

```text
ThreadPlan
DecisionInheritancePacket
ThreadObservation
TuningDecision
ConvergenceStrategy
GenerationalHandoff
ThoughtRecord
```

即使模型不是严格函数调用模式，也可以通过：

```text
结构化提示词
JSON schema 校验
输出修复循环
Observation Contract 二次标注
```

来保证系统可控。

### 3. 工具调用编排

agent+++ 不把工具完全交给模型自由调用。

更稳的链路是：

```text
模型输出执行意图
系统做权限与安全判断
Reflex Layer 调用工具
Observation Contract 回传结果
Thinker 决定下一步
```

DeepSeek 参与完整 agent loop，但不会被设计成无边界工具执行器。

### 4. 长上下文与上下文预算

DeepSeek 的长上下文能力适合长期项目，但 agent+++ 不会把长上下文当垃圾桶。

长上下文的正确用法是：

```text
承载结构化项目状态
承载阶段档案
承载证据摘要
承载当前决策链
保留输出和调频余量
```

不是把所有日志、对话和文件全文都塞进去。

---

## Token 高利用率设计

agent+++ 不是单纯“省 token”，而是追求：

```text
把 token 花在最有决策价值的位置
```

### 1. 原始日志不直接进入 Thinker

工具输出先经过 Observation Contract：

```text
raw output
  -> facts / risks / artifacts / evidence_refs
  -> summary_for_thinker
```

Thinker 读的是决策材料，不是现场噪音。

### 2. 子线程继承决策骨架

每个 Flash 线程不继承完整项目历史，而是继承：

```text
父代目标
判断框架
约束
质量标准
线程范围
完成条件
升级触发条件
```

这就是 Decision Inheritance Packet。

它比“把所有上下文塞给每个子 agent”更省 token，也更防跑偏。

### 3. 项目记忆按相关性析出

Project Memory 不是全部历史，而是从全局沉淀层按当前项目和任务析出：

```text
Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
  -> Thinker Context
```

### 4. 阶段收束后做代际切换

长期项目不靠无限压缩上下文续命：

```text
旧祖 Thinker
  -> Convergence Dock
  -> Generational Handoff
  -> 新祖 Thinker
```

新祖 Thinker 继承阶段结论和边界，不继承完整历史。

一句话：

```text
压缩是把旧材料塞小；
代际切换是只继承结论和边界。
```

### 5. 高保真证据按需打开

Observation Contract 使用 fidelity 策略：

```text
drop
summary
extract
excerpt
raw_window
full_raw
```

默认不给 Thinker 喂完整日志。只有用户要求、验证需要或关键决策需要证据时，才打开高保真窗口。

---

## DeepSeek 不是系统锁定

DeepSeek 是 agent+++ 当前默认模型底座，但架构不绑定单一供应商。

系统层应该抽象为：

```text
Thinker Model
Reflex Model
Observation Model
Memory Extraction Model
```

未来可以替换某个角色的模型，但角色边界不变。

这也是 agent+++ 的重点：

> 真正的竞争力不在“接了哪个模型”，而在“模型被放进了怎样的项目操作系统”。

---

## 风险与边界

### 1. Pro 不能被滥用

如果所有阻塞都升级 Pro，多线程系统会变慢、变贵。

所以需要：

```text
max_secondary_thinkers
max_search_rounds_per_thread
max_replans
max_convergence_rounds
```

### 2. Flash 不能改写父代决策

Flash 可以执行、报告、建议，但不能擅自改变祖 Thinker 的决策框架。

### 3. 长上下文不能替代记忆治理

上下文再长，也需要：

```text
Observation Contract
Global Precipitation Memory
Project Memory View
Generational Handoff
```

否则只是把污染推迟发生。

---

## 总结

agent+++ 选择 DeepSeek，是因为它适合这个系统的核心运行方式：

```text
Pro 负责深度判断
Flash 负责高频执行
Observation Contract 提高反馈质量
Global Precipitation Memory 控制长期记忆
Generational Handoff 控制长期上下文
```

DeepSeek 在 agent+++ 中不是一个普通 provider，而是默认模型底座。

但 agent+++ 的真正目标不是证明某个模型最好，而是证明：

> 当模型被放进正确的思考、执行、反馈和记忆架构里，Agent 才能真正持续推进长期项目。
