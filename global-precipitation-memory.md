# Global Precipitation Memory：全局沉淀层与项目记忆投影

> 日期：2026-06-18  
> 状态：可分享版草稿  
> 所属模块：Project Memory  
> 关联模块：Observation Contract / Thinker / Feedback Bus  
> 用途：说明 agent+++ 的记忆管理机制：全局沉淀、项目析出、创意残渣与遗忘生命周期。

---

## 一句话定义

**Global Precipitation Memory 是 agent+++ 的全局沉淀层：它不是项目记忆本身，而是所有尚未归属的高价值观察、思考记录和用户认知的中转池。**

Project Memory 不是第一沉淀层。

更准确地说：

> Project Memory 是从全局沉淀层按项目语境析出的局部记忆视图。

这个设计把记忆系统从“每个项目各存一份历史”升级成“系统先形成全局认知，再按任务和项目投影回来”。

---

## 价值判断

这套机制有很高价值。

它解决的不是普通 RAG 的“怎么检索更多相关文本”，而是 Agent 长期运行后一定会遇到的四个问题：

1. **记忆污染**：模型思考、工具日志、临时猜测、用户偏好混在一起，越记越脏。
2. **项目割裂**：每个项目各自有记忆，跨项目经验和用户认知无法复用。
3. **长期堆积**：全局记忆只进不出，最后成为高噪声上下文积压层。
4. **创意退化**：长期记忆只服务事实推理，系统越跑越保守、越清醒、越无趣。

Global Precipitation Memory 的核心贡献是：

```text
把记忆从“永久存储”改成“生命周期管理”。
```

一条记忆进入系统后，不是默认永远存在，而是有四种命运：

```text
被项目认领
被用户认知吸收
被创意系统再利用
被遗忘
```

这让 agent+++ 的记忆系统更像认知代谢，而不是资料仓库。

---

## 在四大模块中的位置

agent+++ 的四大核心模块是：

```text
Thinker
Reflex Layer
Feedback Bus
Project Memory
```

Global Precipitation Memory 位于 Project Memory 的上游，但它不是 Project Memory 的子模块。

更合理的位置是：

```text
Tool Output / Pro Thought / Global Conversation
        ↓
Observation Contract
        ↓
Global Precipitation Layer
        ↓
Memory Projection Layer
        ↓
Project Memory View
        ↓
Thinker Context
```

这意味着：

- Observation Contract 负责标注和分类。
- Global Precipitation Layer 负责暂存未归属认知。
- Memory Projection Layer 负责按当前项目和任务析出相关记忆。
- Project Memory View 负责成为 Thinker 当前可用的项目上下文。

---

## 为什么第一沉淀层应该是全局，而不是项目

如果第一沉淀层直接按项目切分，会出现三个问题。

### 1. 用户认知被切碎

用户的表达偏好、决策风格、审美倾向、工作习惯，并不属于某一个项目。

例如：

```text
用户偏好直白链路，优先 shell / Python，而不是复杂抽象。
```

这类信息应该跨项目生效。如果它只被写进某个项目记忆，系统下次换项目时又会忘掉。

### 2. 跨项目经验无法复用

一个项目里的踩坑教训，可能会在另一个项目里产生价值。

例如：

```text
Hook 注册配置修改后不热生效，需要 /clear 或新终端。
```

这不是单项目知识，而是跨项目操作经验。

### 3. 项目记忆会膨胀

如果所有观察都先写 Project Memory，每个项目都会堆积大量临时内容：

- 一次性错误
- 中间猜测
- 已过期设计
- 临时文件状态
- 未被确认的模型解释

Project Memory 应该承担“归属明确的项目资产”，而不是接收所有运行残渣。

---

## Observation Contract 的作用

全局沉淀层里会有大量记忆碎片，必须有机制分类管理。

Observation Contract 正好是这个分类机制。

进入全局沉淀层的内容都应该先经过 Observation Contract 标注：

```text
provenance      来源：tool / model / user / external / mcp
determinism     确定性：deterministic / probabilistic / model_generated
epistemic       证据强度：direct_fact / derived_fact / interpretation / hypothesis / invalid
semantic_role   语义角色：state / result / artifact / risk / preference / decision / question
actionability   行动价值：ignore / audit / context / blocking / user_requested
fidelity        保真策略：summary / extract / excerpt / raw_window / full_raw
```

这些标签决定一条记忆的后续命运：

```text
能不能进入沉淀层？
适合项目析出还是用户认知沉淀？
能不能作为事实上下文？
能不能进入创意残渣池？
什么时候应该遗忘？
```

Observation Contract 因此不只是 Feedback Bus 的输出协议，也是记忆生命周期的分类器。

---

## Pro Thought 也必须进入 Observation Contract

工具输出需要 Observation Contract，Pro 每一轮深度思考后的生成记录也需要。

原因很简单：

> Pro 的思考会产生高价值判断，但也会产生解释、猜测和未确认推断。

如果不经过 Observation Contract，系统很容易把“模型想过的东西”直接沉淀成事实。

所以 Pro Thought Record 进入沉淀层时，默认应该是：

```yaml
source_type: pro_thought
provenance: model
determinism: model_generated
epistemic: interpretation 或 hypothesis
requires_confirmation: true
```

只有当它引用了工具证据、用户确认或项目文档，才可以升级为：

```text
derived_fact
decision
lesson
project_fact
```

这条规则用于避免模型自我污染：

```text
模型猜测 -> 沉淀为记忆 -> 下次作为事实引用
```

这是长期 Agent 记忆系统最危险的闭环之一。

---

## 主链路：从沉淀到项目析出

项目 Pro 思考前，需要从全局沉淀层提取与当前项目相关的内容。

但这不是普通语义检索，而是 Project Relevance Extraction。

```text
Global Precipitation Layer
  + current_project
  + current_task
  + user_intent
  + thinker_phase
        ↓
Relevant Precipitates
        ↓
Project Memory Merge
        ↓
Thinker Context
```

关键点是：

```text
析出的内容先合并到 Project Memory，再与 Project Memory 一起进入 Thinker 上下文。
```

这样 Thinker 看到的是一个统一的项目上下文，而不是两包来源不同的碎片。

---

## 析出后不回流

这套机制必须有一条铁律：

> 从全局沉淀层析出并合并到 Project Memory 的内容，不再作为 Project Memory 的一部分经 Observation Contract 回流到全局沉淀层。

否则会形成记忆回音室：

```text
全局沉淀
  -> 项目记忆
  -> Pro 上下文
  -> Pro 输出
  -> 再次沉淀
  -> 再次析出
```

系统会越记越重复，越想越像。

正确做法是：

```text
active_precipitate
  -> extracted_to_project
  -> compact_tombstone
```

析出后，全局沉淀层删除完整内容，只保留极小 tombstone：

```yaml
memory_id: xxx
content_hash: xxx
extracted_to_project_id: xxx
extracted_at: 2026-06-18
```

tombstone 不进入上下文，不参与创意注入，只负责去重和防回流。

---

## 全局沟通页面：沉淀用户认知

全局沟通页面深度思考后，沉淀的重点不是项目知识，而是用户认知。

例如：

- 用户长期偏好
- 交流风格
- 决策习惯
- 审美倾向
- 对 Agent 行为的期待
- 禁忌和边界
- 反复出现的工作方式

链路是：

```text
Global Conversation
  -> Pro Deep Thinking
  -> Observation Contract
  -> User Cognition Candidate
  -> Global User Memory
```

Global User Memory 与 Project Memory 不同。

Project Memory 关心：

```text
这个项目现在怎么做？
```

Global User Memory 关心：

```text
这个用户长期怎么判断、怎么表达、怎么协作？
```

二者都来自全局沉淀层，但析出目标不同。

---

## 三阶段生命周期

全局沉淀层不能无限增长。

未被项目析出的记忆，应进入生命周期管理：

```text
Retention 保留期
  -> Hallucination 幻觉期
  -> Oblivion 清理期
```

### 1. 保留期

新进入沉淀层的内容先进入保留期。

```yaml
lifecycle_stage: retention
策略: 保留原始结构，不主动加工
用途: 等待项目析出、用户认知沉淀、跨项目复用
```

保留期里的内容仍承担事实责任。

它可以作为项目相关性析出的候选，但必须遵守 Observation Contract 的证据等级。

### 2. 幻觉期

长期未被析出的记忆，不一定立刻删除。

它可以进入幻觉期。

幻觉期的目标不是事实推理，而是创意发散。

```yaml
lifecycle_stage: hallucination
策略: 打碎、去来源化、弱语义重组
用途: 全局沟通、创意发散、命名、类比、写作刺激
```

幻觉期记忆可以被拆成：

- 概念片段
- 隐喻片段
- 结构片段
- 风格片段
- 问题片段
- 未完成想法

例如：

```text
原始记忆：
Observation Contract 是工具输出进入 Thinker 和 Memory 前的结构化协议。

幻觉碎片：
- 观察不是日志
- 事实需要护照
- 进入认知前需要海关
- 记忆沉淀像析晶
```

这些碎片不再承担事实责任，只承担创意刺激责任。

### 3. 清理期

幻觉记忆必须有上限。

超过上限后，低价值碎片进入清理期：

```yaml
lifecycle_stage: oblivion
策略: 低引用、低新颖性、低创意收益、重复内容优先删除
用途: 控制全局沉淀层熵增
```

清理期不是冷藏，而是彻底遗忘。

---

## 幻觉期的安全边界

幻觉期是这套机制最有辨识度的地方，也是最需要边界的地方。

它必须和事实推理硬隔离。

```text
代码修复        禁止注入幻觉记忆
安全判断        禁止注入幻觉记忆
架构定稿        默认禁止，除非用户要求脑暴
事实研究        禁止注入幻觉记忆
创意发散        允许注入
命名写作        允许少量注入
故事化表达      允许注入
```

否则系统会把创意残渣当事实依据。

幻觉期不是让 Agent 生成无依据内容，而是给创意任务提供非线性材料。

---

## 参考数据结构

```python
class PrecipitatedMemory:
    id: str
    content: str
    source_type: str          # tool_observation / pro_thought / global_conversation / user_feedback

    provenance: str
    determinism: str
    epistemic: str
    semantic_role: str
    actionability: str
    fidelity: str

    scope_hint: str           # global / project / user / creative / technical
    project_affinity: dict[str, float]
    user_affinity: float
    confidence: float

    evidence_refs: list[str]
    source_observation_id: str | None
    source_thought_id: str | None

    lifecycle_stage: str      # retention / hallucination / oblivion / extracted
    created_at: str
    last_matched_at: str | None
    extracted_to_project_id: str | None
    tombstone_hash: str | None
```

这个结构的重点不是字段多，而是把三件事分开：

```text
事实可信度
项目相关性
生命周期状态
```

传统记忆系统经常把这三件事混成一个 similarity score。

---

## 最小实现路线

### Phase 1：全局沉淀层 MVP

- Pro Thought / Tool Observation 都经过 Observation Contract。
- 生成 `PrecipitatedMemory`。
- 默认进入 retention。
- 建立 evidence_refs 和 source_id。

### Phase 2：项目析出与合并

- 项目 Pro 思考前，按当前项目和任务做 relevance extraction。
- 相关内容合并进 Project Memory View。
- 析出后全局沉淀层删除完整内容，保留 tombstone。

### Phase 3：生命周期管理

- 未析出的 retention 内容按规则进入 hallucination。
- hallucination 内容拆成创意碎片。
- 超过上限进入 oblivion。

### Phase 4：创意注入

- 只在创意类任务注入 hallucination fragments。
- 技术、事实、安全、代码修复任务默认禁用。
- 记录 creative_yield，用于后续保留或遗忘。

---

## 可实现性评估

整体可实现性：**8/10**。

| 模块 | 可实现度 | 主要难点 |
|---|---:|---|
| Observation Contract 标注所有输入 | 9/10 | Pro thought 不能自我升级为事实 |
| 全局沉淀层 | 9/10 | 数据模型和存储都清晰 |
| 项目相关性析出 | 7/10 | 需要结合项目、任务、阶段，而不是只做向量相似 |
| Project Memory Merge | 8/10 | 需要去重、冲突处理、覆盖规则 |
| 析出后删除与 tombstone | 8/10 | 要防止重复析出和回流 |
| 保留期 / 幻觉期 / 清理期 | 7/10 | 生命周期评分需要调参 |
| 幻觉期创意注入 | 6.5/10 | 必须和事实推理硬隔离 |
| 全局用户认知沉淀 | 8/10 | 需要确认机制和置信度控制 |

最大风险不在工程实现，而在治理规则：

- 模型自我污染
- 全局沉淀层熵增
- 项目记忆和全局沉淀互相回流
- 幻觉碎片误入事实推理

这些风险都可以通过 Observation Contract、生命周期状态、tombstone 和注入场景隔离来控制。

---

## 和 Project Memory 的新关系

原来的说法是：

```text
Observation Contract -> MemoryCandidate -> Project Memory
```

新结构应该改为：

```text
Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

Project Memory 不再是第一写入目标。

它是当前项目从全局沉淀层析出的结构化上下文，是一种项目态投影。

---

## 可分享版总结

Global Precipitation Memory 是 agent+++ 的认知代谢系统。

它让 Observation Contract 不只服务工具反馈，也服务长期记忆治理；让 Project Memory 不再是孤立的项目笔记，而是全局沉淀在项目语境下的投影；让长期未归属的记忆不只是被删除，也可以在创意场景中变成幻觉燃料。

普通记忆系统追求：

```text
记住更多
```

Global Precipitation Memory 追求：

```text
知道什么该归属，什么该发酵，什么该遗忘。
```

这就是它的核心价值。
