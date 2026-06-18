# agent+++ 项目价值点整理

> 日期：2026-06-18  
> 状态：可分享版草稿  
> 用途：单独转发给产品、工程、投资或合作方时，快速说明 agent+++ 的核心价值。  
> 核心版本：四大核心模块版。

---

## 一句话定位

**agent+++ 是一个本地优先的项目教练型 Agent 工作台：用 Thinker、Reflex Layer、Feedback Bus、Project Memory 四大核心模块，把人的碎片想法推进成可执行、可验证、可沉淀的项目资产。**

它不是“开源版 Codex”，也不是“低代码生成器”。agent+++ 的核心不是让模型更会聊天或更会写代码，而是建立一套可靠的人机协作操作系统：用户负责想象、判断和验收，系统负责澄清、执行、反馈、沉淀。

---

## 核心判断

今天大多数 Agent 产品解决的是两类问题：

1. **执行问题**：用户已经知道要做什么，Agent 帮忙写代码、搜资料、跑命令。
2. **对话问题**：用户和 AI 讨论、发散、生成内容。

但真实工作里还有第三类问题更常见：

> 用户并不总是带着完整需求来找 Agent。很多项目是从一句话、一个链接、一个灵感、一次闲聊、一次失败尝试里长出来的。

agent+++ 的价值，是把这条链系统化：

```text
模糊想法
  -> 教练澄清
  -> 产品契约
  -> 技术拆解
  -> 执行验证
  -> 结构化反馈
  -> 项目沉淀
  -> 下次继续
```

这条链的核心，不是单个功能，而是四个模块的边界清晰、协作稳定。

---

## 四大核心模块

### 1. Thinker：思考体

**职责：判断意图、生成契约、制定计划、做方向决策。**

Thinker 对应 Pro 模型和教练/规划管道。它负责高价值判断：

- 评估用户意图成熟度。
- 在 L0-L2 阶段做教练式追问。
- 在 L3 阶段形成用户可判断的 Product Contract。
- 在用户确认后拆成 Technical Contract / Thinking Nodes。
- 根据反馈修正计划，而不是直接读日志硬猜。

Thinker 的核心原则是：

```text
只下达意图和计划，不直接操作工具，不消费原始日志。
```

这让 Pro 模型的注意力留给方向判断，而不是浪费在 `ls`、`git diff`、构建日志和网页噪音里。

---

### 2. Reflex Layer：反射执行层

**职责：快速执行工具、调用 MCP、读写文件、跑命令、收集原始结果。**

Reflex Layer 对应 Flash 模型、工具系统和执行器。它是 agent+++ 的“手和眼”：

- 执行 shell 命令。
- 读写文件。
- 调用本地工具和 MCP。
- 按 Thinking Node 串行或并行推进任务。
- 把结果交给 Feedback Bus，而不是自己做高层方向判断。

Reflex Layer 的核心原则是：

```text
只执行，不越权决策。
```

这带来两个好处：

- 快：低延迟、低成本，适合大量工具动作。
- 稳：执行层不把现场噪音直接污染思考层。

---

### 3. Feedback Bus：反馈总线

**职责：把原始工具输出转成 Thinker 可推理的结构化观察。**

这是 agent+++ 当前最有技术辨识度的模块。

普通 Agent 往往把工具输出原样塞回模型上下文。短任务还能忍，长任务会迅速出现两个问题：

- **上下文污染**：终端输出、文件列表、日志细节淹没关键事实。
- **上下文失真**：如果只做粗暴摘要，又会把关键证据压没。

agent+++ 的 Feedback Bus 不只是压缩器，而是一个 **Observation Contract（观察契约）** 多轴分离器。

它把一次工具输出拆成多个正交维度：

```text
provenance      信息来源：tool / mcp / external / model / user
determinism     确定性：deterministic / probabilistic / model_generated
epistemic       证据强度：direct_fact / derived_fact / interpretation / hypothesis / invalid
semantic_role   语义角色：state / result / error / artifact / risk / question / instruction
actionability   行动价值：ignore / audit / context / blocking / user_requested
fidelity        保真策略：drop / summary / extract / excerpt / raw_window / full_raw
```

这意味着 Feedback Bus 的真正价值不是“省 token”，而是**重建观察**：

```text
原始工具输出
  -> 证据判断
  -> 语义角色判断
  -> 行动价值判断
  -> 保真策略选择
  -> 路由到 Thinker / Memory / UI / Audit / Verifier
```

例如：

| 场景 | 不好的处理 | Observation Contract 的处理 |
|---|---|---|
| `git diff` | 只压成 `+12/-3` | 保留文件、hunk、关键变更片段 |
| `pytest` 失败 | 只说“测试失败” | 保留失败测试名、断言、文件行号 |
| 构建日志 | 全量塞回上下文 | 成功走摘要，失败走关键错误 excerpt |
| Web 搜索 | 简单摘要 | 保留来源、日期、证据片段、可信度 |
| 用户要求看全文 | bypass 后门 | Fidelity Override：有理由、有预算、有范围 |

这里最重要的一点是：旧的 L0-L3 不再是主骨架，而是降级成 `epistemic` 证据强度的一部分。  
“绿门”也不再是绕过系统，而是受控的 `fidelity_override`：

```text
reason      为什么需要高保真
budget      允许多少字符或 token
scope       whole_output / matched_region / file_range
preferred   raw_window / full_raw
```

这让系统既能省上下文，又不牺牲关键证据。

---

### 4. Project Memory：项目记忆

**职责：承接全局沉淀层析出的项目态知识，形成项目树、资产归档和下次继续的上下文。**

Project Memory 不是普通聊天记忆，也不是简单的 `memory.add()`。它处理的是“项目资产”的形成。

更准确地说，Project Memory 不是第一沉淀层。第一沉淀层应该是全局的：

```text
Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

全局沉淀层负责暂存尚未归属的观察、Pro 深度思考记录和用户认知；Project Memory 负责接收其中与当前项目相关、已经被析出的部分。

agent+++ 的信息沉淀管线是：

```text
raw_fragment
  -> candidate_signal
  -> classified
  -> valued
  -> project_matched
  -> confirmed
  -> settled
  -> dismissed
```

它接住三类东西：

- 对话中的灵感、材料、偏好和决策。
- 执行过程中的产物、失败、偏差和教训。
- 用户确认后的项目知识、Product Contract、Thinking Nodes 和 Artifacts。

长期未被项目析出的全局沉淀不会无限堆积，而是进入生命周期管理：

```text
Retention 保留期
  -> Hallucination 幻觉期
  -> Oblivion 清理期
```

保留期用于等待项目析出；幻觉期把长期未归属的记忆打碎成创意残渣，只在创意场景注入；清理期负责彻底遗忘低价值碎片。

这套机制的详细设计见 `global-precipitation-memory.md`。

Project Memory 的核心不是“记住更多”，而是**解释每个项目资产为什么存在**。

```text
Project
  Product Contract      用户确认的需求契约
  Thinking Nodes        Pro 拆解出的计划单元
  Execution Tasks       Flash 执行步骤
  Artifacts             产物文件、文档、配置
  Memories              决策、知识、偏好、踩坑教训
  Inspirations          待确认的灵感碎片
  Decisions             方向选择与取舍原因
```

项目树因此不是 UI 目录，而是工作模型：它解释每个文件、每次执行、每条记忆和哪次判断有关。

---

## 四大模块如何协作

agent+++ 的主循环可以概括成：

```text
用户输入
  -> Thinker 判断意图、澄清需求、形成契约
  -> Reflex Layer 执行工具、读写文件、调用 MCP
  -> Feedback Bus 把原始输出变成结构化观察
  -> Thinker 基于反馈修正计划或继续执行
  -> Project Memory 把长期价值沉淀为项目资产
```

这个架构的关键边界是：

```text
Thinker 不直接碰工具。
Reflex 不做方向判断。
Feedback 不只是摘要，而是观察重建。
Memory 不只是聊天历史，而是项目资产化。
```

这四句话，就是 agent+++ 区别于普通 Agent loop 的地方。

---

## 产品体验价值

### Product Contract 先于 Technical Contract

agent+++ 不要求用户直接给出完美技术需求。用户先确认自己能判断的产品契约：

- 核心功能
- 数据需求
- 行为规则
- 成功标准
- 不做什么

确认后，系统再把它转成 Thinking Nodes 和技术契约。

这让用户专注判断“是不是我要的”，而不是被迫理解代码结构。

### Reality Preview Loop

agent+++ 的默认展示对象不是代码，而是现实结果：

- 可运行页面或工具
- 截图
- 操作演示
- 完成清单
- 偏差摘要
- 风险提示

代码、日志、diff 和测试细节属于开发者/审计视图，不是默认用户视图。

### 本地优先、可审计、可回滚

agent+++ 的信任感来自本地控制：

- 文件系统 JSON 存储，人类可读。
- 工具调用和原始输出可审计。
- 文件编辑有 diff、timeline、rewind。
- shell 命令有风险分类和确认机制。
- 项目记忆、灵感池、任务状态都沉淀在本地。

一句话：

> 用户可以把工作交给 Agent，但不能把控制权丢给 Agent。

---

## 当前完成度判断

从目前项目状态看，agent+++ 已经从“能调工具的桌面 Agent”收敛到“四大核心模块”的架构雏形：

| 核心模块 | 当前状态 | 判断 |
|---|---|---|
| Thinker | 意图成熟度、CoachState、Product Contract、ThinkingNode 已有模型和流程 | 主线成立，体验需要继续打磨 |
| Reflex Layer | tools、executor、serial execution、node runner 已存在 | 执行层雏形成立 |
| Feedback Bus | Observation Contract 多轴分离器已有代码雏形，旧 L0-L3 保持兼容 | 最有架构辨识度 |
| Project Memory | 信息沉淀文档、Project Gravity、Inspiration Pool、项目树方向明确 | 产品价值高，闭环需继续落地 |

横向能力上，项目还需要继续补：

- 任务级恢复和状态机。
- patch-first / diff-first / checkpoint-first 的编辑体验。
- Feedback Bus 到 Project Memory 的自动沉淀连接。
- 任务级 eval，而不只是单元测试和 smoke。
- 更完整的 MCP / 插件生态边界。

但核心价值已经清晰：

> agent+++ 不是一个更会执行命令的 Agent，而是一套把想法、执行、反馈、记忆组织成项目资产的本地工作系统。

---

## 最值得对外强调的三句话

### 1. agent+++ 的核心不是单个 Agent，而是四大模块组成的认知-执行分离系统。

这是架构差异。

### 2. 我们不是简单压缩上下文，而是用 Observation Contract 重建工具观察。

这是技术差异。

### 3. 我们不是让用户看代码，而是让用户判断方向、结果和偏差。

这是体验差异。

---

## 下一步最关键的建设方向

### 1. 把四大核心模块写进主文档

README、ARCHITECTURE、WIKI、MEMORY 需要统一成同一套语言：

```text
Thinker / Reflex Layer / Feedback Bus / Project Memory
```

旧的“碎片到资产”不是废弃，而是归入 Project Memory 的产品主线。  
旧的 L0-L3 不是废弃，而是归入 Feedback Bus 的 epistemic 兼容轴。

### 2. 把 Observation Contract 完整文档化

需要单独形成设计文档，明确：

- 六个核心轴的定义。
- 决策矩阵。
- 示例输入输出。
- Fidelity Override 策略。
- 和旧 L0-L3 / 绿门的迁移关系。

### 3. 建立“反馈不失真”评测

不能只看 token 压缩率。需要任务级评测：

```text
原始工具输出 -> Thinker 是否能正确决策
观察契约摘要 -> Thinker 是否仍能正确决策
```

核心指标是：结构化反馈后的下一步决策是否和原始输出一致。

### 4. 打通 Feedback Bus 到 Project Memory

工具观察中有长期价值的部分，应进入沉淀管线：

```text
Observation -> Decision / Artifact / Lesson / Project Memory
```

这会把“执行过程”也变成项目资产来源。

---

## 可分享版总结

agent+++ 的独特价值，是把 Agent 从“执行命令的工具”推进到“项目形成与沉淀的工作系统”。

它的核心不是一个更强的聊天模型，而是四大模块：

```text
Thinker        负责想清楚
Reflex Layer   负责做得快
Feedback Bus   负责反馈不失真
Project Memory 负责沉淀成资产
```

如果说普通 Agent 的核心问题是“怎么把任务做完”，agent+++ 关心的是更长的一条链：

```text
想法如何被接住？
需求如何变成熟？
执行如何可验证？
反馈如何不污染思考？
结果如何能沉淀？
项目如何能继续？
```

这条链，就是 agent+++ 目前最有价值的地方。
