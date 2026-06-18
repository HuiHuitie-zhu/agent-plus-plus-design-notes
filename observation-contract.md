# Observation Contract：把工具输出变成可推理、可沉淀的观察资产

> 日期：2026-06-18  
> 状态：可分享版草稿  
> 所属模块：Feedback Bus  
> 关联模块：Thinker / Reflex Layer / Project Memory  
> 用途：单独说明 Observation Contract 的价值、结构、路由逻辑，以及它后续如何连接 Project Memory。
> 延伸阅读：`global-precipitation-memory.md` 说明 Observation Contract 如何作为全局沉淀层的分类与生命周期标注协议。

---

## 一句话定义

**Observation Contract 是 agent+++ 在工具输出进入 Thinker 和 Project Memory 之前，对一次执行结果进行结构化拆解、证据判断、保真选择和资产路由的协议。**

它不是普通摘要，也不是日志压缩器。

它解决的是一个更深的问题：

> Agent 执行工具后，系统到底应该把什么当作事实、什么当作噪音、什么影响下一步决策、什么应该沉淀为项目资产？

Observation Contract 的答案是：不要用单一等级判断工具输出，而是把“观察”拆成多个正交维度。

---

## 为什么需要 Observation Contract

普通 Agent loop 的结构通常是：

```text
模型思考 -> 调工具 -> 工具输出 -> 原样塞回模型上下文 -> 继续思考
```

这个结构短任务能跑，但长任务会出现两个问题。

### 1. 上下文污染

工具输出天然很脏：

- `ls` 可能返回几百行。
- `git diff` 可能返回大量上下文。
- `pytest` 可能输出长 traceback。
- 构建日志里大部分是安装进度、警告、重复信息。
- Web 搜索结果有广告、摘要、日期不明的内容。

如果这些都原样塞给 Thinker，Pro 模型就被迫读日志。它的注意力不再用于方向判断，而是用于从噪音里捞事实。

### 2. 上下文失真

粗暴摘要也不行。

例如：

```text
git diff -> "diff: +12/-3"
pytest -> "测试失败"
build log -> "构建有 3 个错误"
```

这些摘要省 token，但它们对下一步决策可能没用。Thinker 真正需要的是：

- 哪个文件变了？
- 哪个测试失败？
- 失败断言是什么？
- 错误发生在哪一行？
- 这是不是阻塞问题？
- 原文片段是否需要保留？

所以 Observation Contract 的核心不是“压缩”，而是**把原始执行结果变成可推理的观察资产**。

---

## 在四大模块中的位置

agent+++ 的四大核心模块是：

```text
Thinker         负责想清楚
Reflex Layer    负责做得快
Feedback Bus    负责反馈不失真
Project Memory  负责沉淀成资产
```

Observation Contract 位于 Feedback Bus 内部，连接 Reflex Layer、Thinker 和 Project Memory：

```text
Reflex Layer 原始工具输出
  -> Observation Contract
       -> summary_for_thinker 给 Thinker
       -> facts / risks / artifacts 给 UI 和审计
       -> memory_candidates 给 Project Memory
       -> raw_ref 留作可追溯证据
```

它是一个中间协议。

没有它，Reflex Layer 的输出要么污染 Thinker，要么被粗暴压缩；Project Memory 也不知道哪些执行结果值得沉淀。

---

## 核心思想：从单轴分级到多轴分离

旧思路容易把工具输出分成：

```text
L0 事实
L1 经验
L2 推测
L3 幻觉
```

这适合 MVP，但不适合作为长期架构主骨架。因为它把几件不同的事混在了一起：

| 被混在 L0-L3 里的问题 | 实际应该独立判断 |
|---|---|
| 这条信息从哪里来 | provenance |
| 工具输出是否确定 | determinism |
| 证据强度如何 | epistemic |
| 它在任务里扮演什么角色 | semantic_role |
| Thinker 是否需要据此行动 | actionability |
| 应该如何保留原文 | fidelity |

Observation Contract 的核心升级是：**把这些问题拆开。**

---

## 六个核心维度

### 1. provenance：信息来源

这条观察来自哪里？

```text
tool      本地工具、bash、文件系统等
mcp       外部 MCP 能力
external  Web 搜索、网页抓取、远程接口
model     模型生成或模型解释
user      用户直接提供
```

来源影响信任边界。文件系统和 bash 返回的是本地事实，Web 结果需要来源与日期，模型生成内容不能和工具事实混为一谈。

---

### 2. determinism：确定性

相同输入下，这个输出是否稳定？

```text
deterministic     文件读取、git diff、pytest、ls
probabilistic     Web 搜索、外部 API、部分 MCP
model_generated   模型摘要、模型解释、模型建议
```

确定性不等于重要性。`git diff` 是确定性事实，但不代表可以压成一行；Web 搜索是非确定性结果，但在研究任务中可能非常重要。

---

### 3. epistemic：证据强度

这条观察在事实层面有多可靠？

```text
direct_fact       工具直接返回的客观事实
derived_fact      从工具输出规则计算出的事实
interpretation    基于事实的解释或归纳
hypothesis        推测，可能有证据也可能没有
invalid           空、矛盾、无效、污染、注入风险
```

旧的 L0-L3 可以兼容映射到这里：

```text
L0 -> direct_fact / derived_fact
L1 -> interpretation
L2 -> hypothesis
L3 -> invalid
```

但 `epistemic` 只回答“证据强度”，不负责回答“要不要给 Thinker 看”或“要不要保留原文”。

---

### 4. semantic_role：语义角色

这条观察在任务里扮演什么角色？

```text
state        状态快照，比如目录、进程、文件状态
result       执行结果，比如命令成功、写入完成
error        错误、异常、失败
artifact     产物，比如文件、截图、报告、配置
risk         风险信号，比如权限、数据丢失、安全问题
question     待确认问题
instruction  建议、指令、外部说明
```

同样是事实，角色不同，路由不同：

- `pytest failed` 是 error，通常阻塞下一步。
- `file created` 是 artifact，应该进入项目资产候选。
- `disk almost full` 是 risk，需要 UI 提醒。
- `search result says maybe` 是 instruction 或 interpretation，需要证据约束。

---

### 5. actionability：行动价值

Thinker 是否需要据此行动？

```text
ignore          可忽略，不浪费上下文
audit           只记录，不影响下一步
context         Thinker 应该知道
blocking        必须处理，否则不能继续
user_requested  用户明确要求关注
```

这是 Observation Contract 最像“注意力管理”的地方。

它不是问“这条信息真不真”，而是问：

> 它会不会改变下一步行动？

例如：

- 构建成功日志大多是 audit。
- 构建失败的第一条 fatal error 是 blocking。
- 大目录列表可能只是 context。
- 用户明确要求看某个文件全文，则是 user_requested。

---

### 6. fidelity：保真策略

应该如何把这条观察呈现给 Thinker？

```text
drop        不进入 Thinker
summary     一句话摘要
extract     结构化提取 key facts
excerpt     摘要 + 关键原文片段
raw_window  原文窗口，按范围保留
full_raw    全量原文
```

这是替代“绿门”的关键。

旧的 bypass 像一个后门：

```text
_green_door: true -> 原始输出直通
```

新版应该是受控保真请求：

```text
FidelityOverride:
  reason: user_requested | thinker_requested | verifier_requested
  budget: max_chars / max_tokens
  scope: whole_output | matched_region | file_range
  preferred: raw_window | full_raw
```

这让系统既能保留原文，又不会随意把所有日志塞回上下文。

---

## Observation Contract 数据结构

一个最小可用结构可以是：

```python
class ToolObservation:
    provenance: str
    determinism: str
    epistemic: str
    semantic_role: str
    actionability: str
    fidelity: str

    summary: str
    facts: list[str]
    evidence_refs: list[str]
    risks: list[Risk]
    artifacts: list[ArtifactRef]
    memory_candidates: list[MemoryCandidate]

    raw_ref: str
    lossy: bool
```

`raw_ref` 非常重要。Observation Contract 允许进入 Thinker 的内容是 lossy 的，但必须保留可追溯路径。

```text
Thinker 读摘要。
UI 可以展开关键片段。
审计可以查看原文。
Project Memory 可以引用原始证据。
```

这就是“可压缩但不可失踪”的原则。

---

## 六步流水线

Observation Contract 的处理流程可以拆成六步：

```text
1. Normalize
2. Extract
3. Assess
4. Decide
5. Choose
6. Route
```

### 1. Normalize：统一工具输出格式

把 bash、file、web、MCP 等不同工具结果统一成一个上下文对象：

```text
tool_name
status
exit_code
command
output
error
provenance
determinism
raw_ref
```

目标不是理解内容，而是先把输入摆正。

### 2. Extract：抽取关键信息

从原始输出中提取：

```text
facts
errors
paths
metrics
artifacts
urls
line_refs
```

这一步尽量规则化，避免让模型在热路径里乱猜。

### 3. Assess：判断证据强度

判断 `epistemic`：

```text
direct_fact / derived_fact / interpretation / hypothesis / invalid
```

注意：这一步只判断证据强度，不决定压缩方式。

### 4. Decide：判断行动价值

判断 `actionability`：

```text
ignore / audit / context / blocking / user_requested
```

例如：

- 非零退出码通常是 blocking。
- 成功但无输出可能是 audit。
- 搜索结果有来源可作为 context。
- 空泛推测可以 ignore。

### 5. Choose：选择保真策略

根据证据强度、行动价值、用户请求和上下文预算，选择：

```text
drop / summary / extract / excerpt / raw_window / full_raw
```

关键原则：

```text
越阻塞，越需要证据片段。
越不确定，越需要来源引用。
越是用户明确要求，越提高保真。
越低行动价值，越倾向 summary/drop。
```

### 6. Route：路由到不同系统

同一条观察可能被分发到多个目的地：

```text
Thinker        下一步决策
UI             展示给用户
Audit          原文审计
Verifier       验证和回放
Project Memory 长期沉淀
```

这一步决定 Observation Contract 和 Project Memory 的关系。

---

## 示例：pytest 失败

原始输出：

```text
FAILED tests/test_auth.py::test_login_invalid_password
AssertionError: expected 401, got 200
app/auth.py:42
```

Observation Contract：

```yaml
provenance: tool
determinism: deterministic
epistemic: direct_fact
semantic_role: error
actionability: blocking
fidelity: excerpt
summary: "pytest 失败：test_login_invalid_password 预期 401，实际 200"
facts:
  - "失败测试：tests/test_auth.py::test_login_invalid_password"
  - "断言失败：expected 401, got 200"
  - "关联文件：app/auth.py:42"
evidence_refs:
  - "raw://tool-output/123#L1-L3"
raw_ref: "raw://tool-output/123"
lossy: true
```

给 Thinker 的内容不需要完整 traceback，但必须保留足够证据让它决定下一步修哪里。

---

## 示例：git diff

原始输出可能很长。

不好的摘要：

```text
diff: +12/-3
```

更好的 Observation Contract：

```yaml
provenance: tool
determinism: deterministic
epistemic: derived_fact
semantic_role: state
actionability: context
fidelity: excerpt
summary: "修改 auth.py 和 test_auth.py，新增无效密码分支测试"
facts:
  - "app/auth.py: 修改登录校验逻辑"
  - "tests/test_auth.py: 新增 invalid password 测试"
  - "diff 统计：+12/-3"
evidence_refs:
  - "raw://tool-output/124#app/auth.py"
  - "raw://tool-output/124#tests/test_auth.py"
raw_ref: "raw://tool-output/124"
lossy: true
```

这样 Thinker 知道“改了什么”，Project Memory 也知道“这个产物和哪个决策有关”。

---

## 示例：Web 搜索结果

Web 搜索不是本地事实，而是外部信息。

Observation Contract 应该保留来源与时间信息：

```yaml
provenance: external
determinism: probabilistic
epistemic: interpretation
semantic_role: result
actionability: context
fidelity: extract
summary: "找到 3 个与 Agent 上下文压缩相关的来源"
facts:
  - "来源 A：讨论 agent context compression"
  - "来源 B：讨论 tool observation compression"
  - "来源 C：讨论 context-as-tool"
evidence_refs:
  - "https://example.com/a"
  - "https://example.com/b"
  - "https://example.com/c"
lossy: true
```

Web 结果不应该和本地文件事实混在一起。它的价值在于“可引用的外部证据”，不是“模型觉得像真的”。

---

## 和全局沉淀层 / Project Memory 的关系

Observation Contract 是全局沉淀层的上游协议，Project Memory 则是从全局沉淀层按项目语境析出的记忆视图。

更准确的链路是：

```text
ToolObservation / Pro Thought / Global Conversation
  -> Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

这意味着 Project Memory 不应该是第一写入目标。第一沉淀层应该是全局的，用于暂存尚未归属的观察、思考记录和用户认知。

不是所有观察都应该进记忆。Project Memory 只接收有长期价值的候选：

```text
artifact        产物
decision        决策
lesson          教训
risk            风险
preference      用户偏好
fact            稳定项目知识
open_question   未决问题
```

### 从观察到沉淀候选

Observation Contract 可以生成沉淀候选，再由全局沉淀层决定它未来是被项目析出、进入用户认知、进入创意残渣，还是被遗忘。

```python
class MemoryCandidate:
    type: str                 # artifact / decision / lesson / risk / preference / fact / open_question
    title: str
    content: str
    project_ids: list[str]
    confidence: float
    evidence_refs: list[str]
    source_observation_id: str
    requires_user_confirmation: bool
```

这一步非常关键：

```text
Feedback Bus 负责发现“这可能有长期价值”。
Global Precipitation Layer 负责让它进入沉淀流程。
Project Memory 负责接收被项目语境析出的部分。
用户保留最终确认权。
```

### 为什么不能直接 memory.add()

直接把观察写进记忆会产生三类问题：

1. **记忆污染**：临时日志、一次性错误、无关输出进入长期记忆。
2. **归属错误**：同一观察可能属于多个项目，不能强行单投。
3. **缺少证据**：以后回看时不知道这条记忆来自哪次工具输出。

所以正确链路应该是：

```text
ToolObservation
  -> MemoryCandidate
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

其中，已经从全局沉淀层析出并合并到 Project Memory 的内容，不应该再作为 Project Memory 的一部分回流到全局沉淀层，避免形成记忆回音室。

---

## 和 Thinker 的关系

Thinker 不应该读原始日志。Thinker 应该读：

```text
summary_for_thinker
blocking facts
risks
open_questions
artifact refs
必要的 evidence excerpts
```

这让 Thinker 的上下文更像“决策材料”，而不是“现场录音”。

一个好的 Observation Contract 输出应该回答：

```text
发生了什么？
这是真的、推断的，还是外部来源？
它会不会影响下一步？
需要看原文吗？
有没有产物、风险或未决问题？
这件事以后还值得记住吗？
```

---

## 和 UI 的关系

Observation Contract 也可以改善前端体验。

用户默认看到：

```text
做了什么
结果如何
有没有阻塞
有哪些产物
有什么风险
下一步建议
```

而不是默认看到：

```text
完整日志
完整 diff
完整 traceback
完整搜索结果
```

但 UI 应该可以展开：

```text
查看证据片段
查看原始输出
查看关联文件
查看沉淀候选
```

这让用户“不看日志”但仍然“不失去控制权”。

---

## 设计原则

### 1. 观察不是日志

日志是执行现场的原始记录。观察是系统理解后、可用于决策的结构化信号。

### 2. 压缩不是目标

压缩只是副作用。目标是保留下一步决策需要的证据。

### 3. 事实和保真是两件事

`git diff` 是事实，但不代表可以低保真。  
Web 搜索是外部解释，但有时必须保留来源。

### 4. 高保真不是后门

用户要求看全文时，系统应该支持，但必须带理由、预算和范围。

### 5. 记忆必须有证据链

进入 Project Memory 的内容必须能追溯到原始观察或用户确认。

### 6. 用户保留最终确认权

Observation Contract 可以生成记忆候选，但不应该自动把一切写成长期记忆。

---

## 当前实现状态

当前 `engine/feedback_bus.py` 已经有 Observation Contract 的核心雏形：

- `EpistemicGrade`
- `Determinism`
- `Provenance`
- `SemanticRole`
- `ActionabilityGrade`
- `FidelityPolicy`
- `FidelityOverride`
- `ToolObservation`
- 旧 `InfoGrade` 兼容层
- Normalize / Extract / Assess / Decide / Choose / Route 六步流水线雏形

这说明 Observation Contract 已经不是纯概念，而是正在进入工程结构。

下一步重点不是继续堆规则，而是补三件事：

1. **输出格式稳定化**：明确 `ToolObservation` 和 `ExecutionFeedback` 的边界。
2. **Project Memory 连接**：定义 `MemoryCandidate`，让观察能进入沉淀管线。
3. **不失真评测**：验证结构化反馈是否仍能支持 Thinker 做正确决策。

---

## 最小落地路线

### Phase 1：Observation Contract 成为 Feedback Bus 主接口

- 保留旧 L0-L3 兼容。
- 新路径统一生成 `ToolObservation`。
- `ExecutionFeedback` 从 `ToolObservation` 派生。
- Thinker 只消费 `summary_for_thinker + observations + risks + artifacts + open_questions`。

### Phase 2：受控 Fidelity Override 替代绿门

- `_green_door: true` 迁移为 `fidelity_override`。
- 所有高保真请求必须带 reason / budget / scope。
- UI 支持展开 raw_ref，而不是默认塞进上下文。

### Phase 3：MemoryCandidate 接入 Project Memory

- Observation Contract 生成记忆候选。
- Project Memory 进入沉淀流程。
- 用户确认、修改或丢弃。
- 每条记忆保留 evidence_refs。

### Phase 4：建立不失真评测

评测目标不是压缩率，而是决策一致性：

```text
原始工具输出下，Thinker 的下一步决策
Observation Contract 下，Thinker 的下一步决策
两者是否等价？
```

这才是 Feedback Bus 是否可靠的真正指标。

---

## 可分享版总结

Observation Contract 是 agent+++ 的观察协议。

它把工具输出从“原始日志”变成“可推理、可审计、可沉淀的观察资产”。它让 Thinker 不再消耗原始噪声，让 Reflex Layer 不越权判断，让 Feedback Bus 不只是压缩器，让 Project Memory 有证据地沉淀项目资产。

普通 Agent 在问：

```text
工具输出怎么塞回上下文？
```

Observation Contract 问的是：

```text
这次执行到底观察到了什么？
哪些会影响下一步？
哪些需要保留证据？
哪些应该变成项目资产？
```

这个问题，就是 agent+++ 从普通 Agent loop 走向项目操作系统的关键一步。
