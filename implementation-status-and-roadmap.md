# agent+++ 实现状态与路线图

> 日期：2026-06-19  
> 状态：可分享版草稿  
> 用途：说明 agent+++ 当前已经完成了哪些工程验证、哪些仍是目标架构，以及下一阶段如何从可运行内核走向产品化系统。

---

## 一句话结论

**agent+++ 还不是完整产品，但已经不是纯概念设计。**

目前项目已经完成了若干关键可行性证明：模型接入、上下文预算、工具安全、反馈总线、Observation Contract 原型、项目数据结构和部分执行链路。当前公开仓库先发布设计笔记；对应代码和测试仍处在整理阶段，后续会以更干净的 public kernel 形式逐步开放。

这篇文档只描述公开层面的工程状态，不包含敏感凭据、内部部署细节或不可公开的实现信息。

---

## 当前已经能证明什么

agent+++ 当前最重要的进展，不是“功能很多”，而是几个核心假设已经有工程支撑。

### 1. 模型底座可以被角色化使用

项目已经具备 DeepSeek Native Client、模型路由和成本估算相关能力。  
这说明 agent+++ 可以把模型从“单一聊天接口”拆成不同角色：

```text
Flash 负责高频执行、反馈处理和轻量判断。
Pro 负责深度思考、项目决策、阶段收束和代际交接。
```

这为后续 Thinker / Reflex Layer 分工提供了基础。

### 2. 上下文不是只能被动压缩

Context Budget 相关实现证明了一个关键方向：

```text
上下文管理不是“尽量塞更多”，而是决定什么应该进入哪一层上下文。
```

这和 agent+++ 的长期目标一致：原始工具日志、项目记忆、用户偏好、执行证据和 Thinker 决策不应该混在同一个上下文里。

### 3. 工具执行需要安全边界

项目已经有工具安全、命令分类和风险判断相关实现。  
这说明 Reflex Layer 不是简单“能跑命令”就够了，而必须能回答：

- 这个动作是不是只读？
- 是否可能修改用户文件？
- 是否需要确认？
- 是否可以并发执行？
- 是否应该被审计？

这也是本地优先 Agent 的基础信任来源。

### 4. Feedback Bus / Observation Contract 的核心假设成立

当前项目已经有 Feedback Bus 和 Observation Contract 原型，能够把原始工具输出转成结构化观察。

这是 agent+++ 最关键的工程验证之一：

```text
raw output
  -> provenance
  -> determinism
  -> epistemic
  -> semantic_role
  -> actionability
  -> fidelity
  -> routing
```

它证明 agent+++ 不只是做日志摘要，而是在尝试重建“观察”本身。  
Thinker 未来不应该直接读原始噪音，而应该读取可解释、可路由、可保真的结构化反馈。

### 5. 项目数据层和工作台雏形已经存在

项目已经有 project、task、session、memory、preview、workbench 等相关模块。  
这些能力说明 agent+++ 可以从一次性对话，转向长期项目工作：

```text
Project
  -> Session
  -> Task
  -> Thinking Node
  -> Observation
  -> Artifact
  -> Memory
  -> Preview
```

这还不是完整 Project Memory，但已经有承接项目化工作的基础。

---

## 实现状态矩阵

> 说明：下表中的“工程证据”指当前开发仓库中的模块和测试，不代表这些文件已经全部进入公开设计笔记仓库。公开版本会先保持设计说明和路线图，后续再选择低敏、可验证、能代表架构价值的模块逐步开放。

| 能力 | 当前状态 | 当前工程证据 | 公开证明计划 |
|---|---|---|---|
| DeepSeek Native Client | 已有基础实现 | 模型客户端和客户端测试 | 后续公开模型接入抽象，不包含本地环境参数 |
| Model Router | 已有基础实现 | 模型路由和路由测试 | 后续公开角色路由策略和降级样例 |
| Context Budget | 已有基础实现 | 上下文预算模块和上下文测试 | 后续公开最小预算策略和测试用例 |
| Tool Safety | 已有基础实现 | 命令分类、安全检查和安全测试 | 后续公开只读/写入/高风险动作的判定样例 |
| Feedback Bus | 原型已成立 | Feedback Bus 模块和复杂输出测试 | 优先开放 Observation Contract public kernel |
| Observation Contract | 原型已成立 | 公开设计文档、Feedback Bus 测试 | 补足运行时字段、评测集和可复现实验 |
| Reflex Layer | 部分实现 | 执行器和任务执行相关测试 | 后续公开标准 Thread Task 输入输出协议 |
| Thinker | 设计明确，工程早期 | Thinker 设计文档、planner / decomposer 雏形 | 先公开 Decision Inheritance Packet 规格，再开放 MVP |
| Project Memory | 基础数据层已有，闭环未完成 | memory / project API 雏形 | 后续公开项目记忆投影和沉淀析出样例 |
| Global Precipitation Memory | 设计明确，待实现 | 公开设计文档 | 先公开生命周期模型，再实现 Retention / Hallucination / Oblivion |
| Workbench UI | 原型已有 | workbench / preview 原型 | 后续公开 Thinker 线程、阶段收束和证据视图 |
| Evaluation | 有初始实验 | Observation Contract sufficiency experiment | 建立决策保持率、反馈保真率、任务完成率评测 |

---

## 如何验证

当前公开设计笔记仓库主要提供架构说明，尚不要求外部读者直接运行代码。短期验证方式分两层：

1. **公开可读验证**：阅读 Observation Contract、Global Precipitation Memory、Thinker Architecture 和本路线图，确认四大模块边界是否自洽。
2. **后续可运行验证**：当 public kernel 发布后，通过最小测试集验证模型接入抽象、上下文预算、安全检查和结构化反馈。

计划中的最小公开测试集会覆盖：

```text
Feedback Bus / Observation Contract
Context Budget
Model Client / Model Router
Tool Safety
```

这些测试不用于证明完整产品已经完成，而用于证明当前可运行内核的关键假设：反馈可以被结构化、上下文可以被预算管理、工具动作可以被安全分类、模型角色可以被分离。

---

## 现在不应该夸大的部分

为了保持项目可信，下面这些能力目前不应该被描述成“已完整实现”：

- Thinker 还不是完整的项目决策系统。
- 多 Flash Agent 协同还需要标准线程协议。
- Ancestor Thinker 的调频和收束站仍处在架构设计阶段。
- Global Precipitation Memory 还没有形成完整运行时闭环。
- Project Memory 还没有和沉淀层析出机制完全打通。
- Observation Contract 已有原型，但字段、路由和评测还需要继续补齐。

这不是坏消息。恰恰相反，这说明项目边界是清楚的：哪些已经验证，哪些正在搭桥，哪些属于下一阶段。

---

## 下一阶段路线图

### Phase 1：巩固可运行内核

目标是让当前已经存在的能力更稳定：

- 固化 DeepSeek Client、Model Router、Context Budget 的边界。
- 补全 Feedback Bus 的结构化字段。
- 增加 Observation Contract 的回归测试。
- 把安全策略、执行器、反馈总线之间的接口收紧。

这一阶段的交付物不是新概念，而是稳定的工程底盘。

### Phase 2：把 Reflex Layer 标准化

目标是让 Flash 执行层变成可复用的线程执行单元：

- 定义 Thread Task 输入输出协议。
- 区分 read / write / destructive / concurrent-safe 任务。
- 让执行结果统一进入 Feedback Bus。
- 为失败任务生成可交给次级 Thinker 的问题包。

这一阶段完成后，agent+++ 才能真正开始多线程执行。

### Phase 3：实现 Thinker MVP

目标是做出最小可用的项目决策链：

```text
Ancestor Thinker
  -> Thread Plan
  -> Decision Inheritance Packet
  -> Reflex Execution
  -> Thread Observation
  -> Tuning Decision
```

Thinker MVP 的关键不是“更会规划”，而是能在子线程完成后，根据结构化观察进行调频。

### Phase 4：实现 Convergence Dock

目标是让项目具备阶段性停靠站：

- 手动触发阶段收束。
- Ancestor Thinker 判断触发阶段收束。
- 项目结束触发总收束。
- 整理产出、运行测试、检查偏移。
- 生成下一代 Thinker 的干净决策链。

Convergence Dock 的价值，是防止长期项目越跑越偏，同时避免原始父辈上下文长期污染后续进程。

### Phase 5：实现 Global Precipitation Memory MVP

目标是把记忆从“项目内存储”升级为“全局沉淀、项目析出”：

```text
Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

这一阶段需要完成：

- 全局沉淀层分类。
- 项目相关性析出。
- 析出后 tombstone / delete 机制。
- Retention / Hallucination / Oblivion 生命周期。
- Pro 深度思考记录进入沉淀层前的 Observation Contract 标注。

### Phase 6：建立产品化评测体系

agent+++ 需要评测的不只是“测试是否通过”，还包括：

- 结构化反馈后的下一步决策是否保持正确。
- 多线程执行是否比单 Agent 更快。
- Thinker 调频是否减少返工。
- Convergence Dock 是否降低项目偏移。
- Project Memory 是否提升长期项目继续能力。
- Token 是否花在决策上，而不是日志噪音上。

---

## 产品化判断

如果按当前路线推进，agent+++ 的产品化核心不是“做一个更强的聊天窗口”，而是形成一个本地项目操作系统：

```text
Thinker        决策
Reflex Layer   执行
Feedback Bus   观察
Memory          沉淀
Workbench       验收
```

它的竞争力来自三个组合优势：

1. **认知-执行分离**：Pro 不被工具噪音淹没，Flash 不承担高层方向判断。
2. **反馈不失真**：Observation Contract 让工具输出成为可推理的观察，而不是随手摘要。
3. **长期项目连续性**：Project Memory 和全局沉淀层让项目可以阶段性继续，而不是每次从上下文压缩里捞残片。

---

## 公开口径

对外可以这样描述当前进度：

> agent+++ is currently at the runnable-kernel stage. The project has working foundations for model access, context budgeting, safety checks, structured feedback, project/task data, and workbench prototypes. The next milestone is to connect these foundations into a Thinker-led multi-thread execution loop with Observation Contract and Memory as first-class system components.

中文口径：

> agent+++ 当前处在可运行内核阶段。项目已经具备模型接入、上下文预算、安全检查、结构化反馈、项目/任务数据和工作台原型等基础能力。下一阶段目标，是把这些基础能力连接成由 Thinker 主导的多线程执行闭环，并把 Observation Contract 和 Memory 提升为一等系统组件。

---

## 总结

agent+++ 现在最值得分享的，不是“已经完成了一个完美 Agent”，而是：

```text
核心架构已经清晰；
关键工程假设已经开始被代码验证；
下一阶段路线能从现有模块自然生长出来。
```

这就是当前项目最好的公开状态：不夸张，但有骨架；不泄密，但能证明方向；不装完成，但已经能看见产品形状。
