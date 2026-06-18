# Thinker Architecture：项目决策拆解大师

> 日期：2026-06-18  
> 状态：实操设计稿  
> 所属模块：Thinker  
> 关联模块：Observation Contract / Feedback Bus / Reflex Layer / Global Precipitation Memory / Project Memory  
> 用途：整理 agent+++ Thinker 的确定架构、关键协议、落地路线和可参考资料，供后续工程实操使用。

---

## 一句话定义

**Thinker 是 agent+++ 的项目决策拆解大师：它负责用 Pro 模型完成深度判断，把复杂项目拆成可并行推进的多线程任务，并在子线程执行过程中持续调频，保证速度、质量和方向一致。**

Thinker 不是普通 planner。

它不是简单输出 todo list，也不是一个单轮 Pro 调用器。它的核心价值是：

```text
深度决策
  -> 多线程拆解
  -> 父代决策继承
  -> Flash 并行执行
  -> 局部阻塞升级
  -> 祖 Thinker 调频
  -> 结果合并与收束
```

这套结构让 agent+++ 可以做到：

```text
Pro 做最深判断
Flash 做最大吞吐
次级 Thinker 只处理局部难点
祖 Thinker 负责全局质量和方向
```

这就是“比普通 Agent 想得深，但仍然更快”的核心内核。

---

## 架构定位

Thinker 位于 agent+++ 四大核心模块的决策中心：

```text
User Intent
  ↓
Thinker
  ↓
Reflex Layer
  ↓
Feedback Bus / Observation Contract
  ↓
Thinker 调频
  ↓
Global Precipitation Memory / Project Memory
```

它只消费结构化上下文，不直接读原始工具日志：

```text
Project Memory View
Projected Global Memory
Observation Contract
Execution Feedback
Open Questions
User Intent
```

它输出的是结构化决策：

```text
Product Contract
Technical Contract
Thread Plan
Decision Inheritance Packet
Execution Intent
Tuning Decision
Convergence Strategy
Generational Handoff
Thought Record
```

---

## Thinker 的核心职责

### 1. 决策拆解

祖 Thinker 读取用户目标、项目记忆和当前观察，把复杂任务拆成多个可执行线程。

输出不是普通任务列表，而是一个带依赖和风险的线程计划：

```text
Thread Plan / DAG
```

### 2. 多线程协作

可直接执行的线程交给 Flash agents 并行推进。

有问题的线程不阻塞全局：

```text
可做线程继续跑
有问题线程先搜索
搜索仍不确定则进入次级 Thinker
次级 Thinker 想清楚后继续交给 Flash 执行
```

### 3. 父代决策继承

每个子线程启动前，必须继承父代的决策骨架。

子代不继承父代完整上下文，而是继承：

```text
父代目标
父代判断框架
已接受假设
已排除方案
质量标准
约束边界
当前线程范围
升级触发条件
```

### 4. 祖级调频

祖 Thinker 不是发完任务就消失。

每个子线程节点结束后，都要通过 Observation Contract 回传结果。祖 Thinker 根据这些结果判断：

```text
继续
合并
搜索
升级次级 Thinker
重排计划
停止线程
询问用户
```

这就是项目质量控制的核心。

### 5. 反思沉淀

Thinker 每轮深度思考结束后，要生成 Thought Record。

Thought Record 不暴露完整内部推理，而是结构化记录：

```text
decision
rationale_summary
assumptions
rejected_options
open_questions
memory_candidates
confidence
evidence_refs
```

然后交给 Observation Contract 标注，进入 Global Precipitation Memory。

---

## 主运行链路

```text
1. Project Context Compose
   组装 Project Memory View、全局析出、用户输入、近期 Observation。

2. Ancestor Thinker Reasoning
   Pro 深度思考，确定目标、约束、质量标准和拆解策略。

3. Decision Decomposition
   输出 Thread Plan / DAG。

4. Inheritance Build
   为每个线程生成 Decision Inheritance Packet。

5. Parallel Flash Execution
   可执行线程并行跑。

6. Thread Observation
   每个线程结束后生成 Observation Contract。

7. Tuning Checkpoint
   祖 Thinker 读取线程观察并调频。

8. Local Escalation
   阻塞线程可先搜索，再进入次级 Thinker。

9. Merge & Replan
   合并结果，检测冲突，必要时重排 DAG。

10. Convergence Dock
    在手动触发、祖 Thinker 判断或项目结束时进入收束停靠站，整理产出、合并结果、调用工具做项目测试。

11. Generational Handoff
    项目继续时，由停靠 Thinker 生成新的决策链，接替原始祖 Thinker 成为下一阶段祖辈 Thinker。

12. Thought Record
    祖 Thinker 生成本轮决策记录，进入 Observation Contract 和全局沉淀层。
```

---

## 关键协议 1：Thread Plan

Thread Plan 是 Thinker 的任务拆解结果。

它应该是 DAG，而不是线性 todo。

```python
class ThreadPlan:
    thread_id: str
    parent_thinker_id: str
    goal: str
    scope: str
    dependencies: list[str]
    can_start_now: bool

    executor_type: str
    # flash / secondary_thinker / search_then_flash / human_confirm

    required_context: list[str]
    expected_artifacts: list[str]
    done_condition: str

    risk_level: str
    escalation_triggers: list[str]
    merge_policy: str
```

Thread Plan 要回答：

```text
这个线程负责什么？
依赖谁？
现在能不能开始？
应该由 Flash 做，还是先搜索/升级？
完成标准是什么？
出问题时怎么回报？
结果怎么合并？
```

---

## 关键协议 2：Decision Inheritance Packet

Decision Inheritance Packet 是父代决策继承包。

这是 Thinker 架构里最关键的协议之一。

子线程不能自己重新理解整个项目，也不能拿到过大的上下文包。它必须继承一份压缩后的父代决策骨架：

```python
class DecisionInheritancePacket:
    parent_thinker_id: str
    parent_goal: str
    parent_decision: str
    decision_frame: str

    accepted_assumptions: list[str]
    rejected_options: list[str]
    constraints: list[str]
    quality_bar: list[str]

    thread_scope: str
    forbidden_scope: list[str]
    dependencies: list[str]
    done_condition: str

    escalation_triggers: list[str]
    return_contract: str
    evidence_requirements: list[str]
```

设计原则：

```text
子代继承父代的判断边界，不继承父代的完整上下文。
```

这样既能防跑偏，又不会把每个子线程都拖成巨大 Pro 上下文。

---

## 关键协议 3：Thread Observation

每个 Flash 线程或次级 Thinker 线程结束时，不能只返回 done / failed。

它必须返回 Thread Observation，并经过 Observation Contract：

```python
class ThreadObservation:
    thread_id: str
    parent_thinker_id: str
    status: str
    # success / blocked / partial / failed / needs_review

    completed_facts: list[str]
    artifacts: list[str]
    risks: list[str]
    blockers: list[str]
    deviations_from_plan: list[str]

    confidence: float
    evidence_refs: list[str]
    needs_escalation: bool
    suggested_next_action: str
```

Observation Contract 负责给这些返回打标：

```text
provenance
determinism
epistemic
semantic_role
actionability
fidelity
```

祖 Thinker 只消费这些结构化观察，不读线程原始日志。

---

## 关键协议 4：Tuning Decision

祖 Thinker 在每个 checkpoint 做调频。

```python
class TuningDecision:
    checkpoint_id: str
    parent_thinker_id: str
    target_thread_id: str

    action: str
    # continue / merge / search / escalate / replan / stop / ask_user

    reason: str
    updated_constraints: list[str]
    updated_dependencies: list[str]
    new_threads: list[ThreadPlan]
    merge_notes: list[str]
```

调频判断维度：

```text
是否符合父级计划？
是否产生新风险？
是否需要补证据？
是否需要局部 Pro 思考？
是否能合并进主线？
是否与其他线程冲突？
是否触发用户确认阈值？
```

---

## 关键协议 5：Convergence Strategy

Convergence Strategy 是 Thinker 的收束策略。

前面的 Tuning Decision 解决的是节点级调频：

```text
某个线程结束后，下一步该继续、升级、合并还是停止？
```

Convergence Strategy 解决的是阶段级和项目级收束：

```text
多个线程跑了一段后，项目是否需要停靠、整理、测试、校准方向？
```

它对应一个特殊 Thinker 模式：

```text
Convergence Thinker / Docking Thinker
```

这个 Thinker 可以调动工具。它不是普通执行层，也不是普通祖 Thinker 思考层，而是项目停靠站：

```text
收集所有线程结果
整理阶段产出
合并项目状态
调用测试/验证工具
检查是否跑偏
决定继续、补线、返工、结束
```

参考结构：

```python
class ConvergenceStrategy:
    convergence_id: str
    parent_thinker_id: str
    trigger: str
    # manual / ancestor_judged / phase_end / project_end / risk_threshold / drift_detected

    scope: str
    # phase / project / thread_group

    inputs: list[str]
    # thread observations, artifacts, project memory view, open risks

    consolidation_tasks: list[str]
    validation_tasks: list[str]
    test_commands: list[str]
    acceptance_criteria: list[str]

    allowed_tools: list[str]
    output_contract: str

    decision: str
    # continue / replan / add_threads / fix_before_continue / ready_for_user_review / finish

    handoff_required: bool
    next_generation_scope: str | None
    evidence_refs: list[str]
```

### 触发方式

收束可以由三种方式触发：

```text
1. 手动触发
   用户觉得该停一下，要求整理、测试、验收。

2. 祖 Thinker 判断触发
   多个线程完成、风险升高、方向漂移、合并冲突、质量不足。

3. 项目/阶段自然结束触发
   phase_end 或 project_end，进入总收束或阶段收束。
```

### 阶段收束

阶段收束用于项目中途停靠：

```text
多线程执行一轮
  -> 阶段收束
  -> 整理产出
  -> 合并项目状态
  -> 跑阶段测试
  -> 修正 Thread Plan
  -> 继续下一阶段
```

适合：

- 多个线程已经产生产物。
- 有线程完成，有线程阻塞。
- 计划开始出现偏移。
- 需要阶段性验收。
- 继续跑之前需要统一上下文。

### 总收束

总收束用于项目结束前：

```text
所有关键线程完成
  -> 总收束
  -> 汇总产出
  -> 项目级测试
  -> 验证完成条件
  -> 整理交付物
  -> 生成最终 Thought Record
  -> Project Memory / Global Precipitation 更新
```

总收束不是“写总结”，而是项目停靠验收。

它必须回答：

```text
最初目标是否完成？
产物在哪里？
测试是否通过？
哪些风险仍然存在？
哪些决策被改变过？
哪些记忆需要沉淀？
用户是否需要最终确认？
```

### 项目继续时的代际切换

收束站处理好后，如果项目还要继续，不应该让原始祖 Thinker 无限持有上下文继续跑。

正确策略是：

```text
旧祖 Thinker
  -> Convergence Dock
  -> 阶段总结 / 项目状态快照 / 新决策链
  -> 新祖 Thinker
  -> 下一阶段 Thread Plan
```

停靠 Thinker 在这里承担代际切换角色：

```text
把上一阶段的杂乱执行历史收束成干净的阶段记录；
把仍然有效的目标、约束、决策和风险整理成新的父代决策骨架；
生成下一阶段的决策链，作为新祖 Thinker 的起点。
```

这样可以避免两个问题：

1. **原始父辈上下文污染**

长期项目跑久以后，原始祖 Thinker 的上下文里会混有大量过期判断、已解决阻塞、废弃方案和临时执行细节。

如果一直继承这个上下文，后续阶段会越来越慢，也越来越容易被旧信息拖偏。

2. **长期项目缺少阶段记录**

如果项目一直滚动执行，没有停靠总结，最后只会留下大量线程日志和零散产物。

代际切换强制生成阶段记录：

```text
本阶段做成了什么？
哪些决策改变了？
哪些线程被放弃？
当前项目状态是什么？
下一阶段目标是什么？
新祖 Thinker 应该继承哪些边界？
```

这让长期项目天然形成完整阶段档案。

---

## 关键协议 6：Generational Handoff

Generational Handoff 是 Thinker 的代际交接协议。

它发生在阶段收束或总收束之后，且项目需要继续推进时。

```python
class GenerationalHandoff:
    previous_ancestor_thinker_id: str
    new_ancestor_thinker_id: str
    convergence_id: str

    phase_summary: str
    project_state_snapshot: str
    completed_artifacts: list[str]
    validated_results: list[str]
    unresolved_risks: list[str]
    open_questions: list[str]

    inherited_goals: list[str]
    inherited_constraints: list[str]
    inherited_decisions: list[str]
    deprecated_assumptions: list[str]
    rejected_threads: list[str]

    next_phase_goal: str
    next_decision_frame: str
    next_quality_bar: list[str]
    recommended_thread_plan: list[ThreadPlan]

    evidence_refs: list[str]
    memory_candidates: list[str]
```

设计原则：

```text
新祖 Thinker 继承阶段结论，不继承完整阶段历史。
```

Generational Handoff 要输出两类东西：

1. **阶段档案**

用于 Project Memory：

```text
阶段目标
阶段产出
验证结果
关键决策
废弃路线
未解决风险
下一阶段入口
```

2. **新父代决策骨架**

用于下一阶段祖 Thinker：

```text
next_phase_goal
next_decision_frame
inherited_constraints
next_quality_bar
recommended_thread_plan
```

这让项目继续时的上下文更短、更干净、更快。

### 为什么收束 Thinker 可以调动工具

普通祖 Thinker 不直接执行工具，是为了避免 Pro 模型陷入日志和细节。

但收束阶段不同。

收束 Thinker 需要做项目验收，必须可以调动受控工具：

```text
运行测试
读取产物
检查文件状态
生成报告
对比完成条件
验证关键路径
```

它仍然不应该自由执行任意工具，而应通过收束策略限制：

```text
allowed_tools
test_commands
validation_tasks
acceptance_criteria
```

这让收束 Thinker 成为项目停靠站，而不是失控执行器。

### 收束站不是终点，而是交接点

收束站可以结束项目，也可以让项目进入下一代决策链。

```text
如果完成条件满足：
  final_convergence -> user_review / finish

如果项目继续：
  phase_convergence -> generational_handoff -> new_ancestor_thinker
```

这个设计让 Thinker 不需要无限上下文续命，而是通过阶段快照持续接力。

---

## 线程状态机

```text
planned
  ↓
ready
  ↓
running_flash
  ↓
observed
  ↓
checkpoint_review
  ├── continue
  ├── merge
  ├── search
  ├── escalate_to_secondary_thinker
  ├── replan
  ├── ask_user
  └── stop
        ↓
convergence_dock
        ↓
validated / replan / finish
```

阻塞线程的推荐路径：

```text
blocked
  -> search_once
  -> still_unclear
  -> secondary_thinker
  -> updated_decision_packet
  -> flash_continue
```

这样局部问题不会拖停整个项目。

项目级状态机：

```text
planning
  -> parallel_execution
  -> checkpoint_tuning
  -> phase_convergence
  -> generational_handoff
  -> new_ancestor_thinker
  -> next_phase 或 replan
  -> final_convergence
  -> user_review / finish
```

---

## 协作纪律

### 1. Flash agent 不能改写父级决策

Flash agent 可以：

- 执行任务
- 收集事实
- 生成产物
- 报告风险
- 提出建议

Flash agent 不可以：

- 擅自改变父级目标
- 擅自扩大任务范围
- 擅自推翻祖 Thinker 的 decision frame
- 把局部发现直接升级为全局决策

全局决策只能由祖 Thinker 或授权的次级 Thinker 修改。

### 2. 次级 Thinker 只能处理局部难点

次级 Thinker 的职责是：

```text
解决某个线程中的不确定性。
```

它不能接管整个项目。

次级 Thinker 输出后，应回到祖 Thinker 的调频机制里。

### 3. 每个线程必须有 done_condition

没有完成条件的线程不能启动。

否则并行系统会失去边界，难以判断是否真正完成。

### 4. 每次合并必须保留证据

合并不是把文本拼起来。

合并结果必须说明：

```text
哪些线程贡献了什么？
是否有冲突？
采用了哪个结果？
为什么？
证据在哪里？
```

### 5. 多线程项目必须设置收束点

多线程系统不能只靠节点完成推进。

每个阶段都应该预设收束点：

```text
phase convergence checkpoint
final convergence checkpoint
```

没有收束点的多线程执行，很容易产生局部都对、整体跑偏的结果。

### 6. 收束 Thinker 可以调工具，但必须受控

收束 Thinker 是项目停靠站，可以调用验证工具和测试工具。

但它的工具权限应由 Convergence Strategy 显式声明：

```text
允许跑哪些测试？
允许读哪些文件？
允许生成哪些报告？
是否允许写入修复？
是否需要用户确认？
```

### 7. 长期项目必须代际切换

长期项目不能让同一个祖 Thinker 无限延续。

每次重要阶段收束后，应生成 Generational Handoff：

```text
旧祖 Thinker 退场
阶段档案进入 Project Memory
新祖 Thinker 继承干净决策骨架
下一阶段重新拆解 Thread Plan
```

这比持续压缩上下文更干净。

压缩是把旧材料塞小；代际切换是只继承结论和边界。

---

## 预算与质量门

Thinker 必须有预算控制，否则多 agent 会失控。

建议第一版就支持：

```text
max_threads
max_parallel_threads
max_secondary_thinkers
max_search_rounds_per_thread
max_replans
max_convergence_rounds
max_generational_handoffs
max_tool_calls_per_thread
quality_floor
timeout_per_thread
```

推荐默认策略：

```text
普通任务：2-4 个 Flash 线程
复杂项目：4-8 个 Flash 线程
搜索任务：每线程最多 1-2 轮搜索
次级 Thinker：默认最多 1-2 个
祖 Thinker 调频：节点结束后 checkpoint，不做高频实时调频
阶段收束：每个主要 phase 至少一次
总收束：项目结束前必须一次
代际切换：长期项目每次阶段收束后默认生成
```

---

## 和 Observation Contract 的关系

Observation Contract 是 Thinker 的眼睛。

Thinker 不应该读原始工具输出，而应该读：

```text
Thread Observation
Execution Feedback
facts
risks
artifacts
blockers
evidence_refs
```

Thinker 每轮输出也要反向进入 Observation Contract：

```text
Thought Record
  -> Observation Contract
  -> Global Precipitation Memory
```

这让 Thinker 的高价值判断能够沉淀，但不会把模型猜测直接当事实。

---

## 和 Global Precipitation Memory 的关系

项目 Pro 思考前：

```text
Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory Merge
  -> Thinker Context
```

项目 Pro 思考后：

```text
Thought Record
  -> Observation Contract
  -> Global Precipitation Layer
```

但有一条铁律：

```text
已经析出并合并到 Project Memory 的内容，不再作为 Project Memory 的一部分回流到全局沉淀层。
```

否则会形成记忆回音室。

---

## 可完成性评估

整体可完成性：**7.5/10**。

| 模块 | 可完成性 | 判断 |
|---|---:|---|
| 祖 Thinker 拆解多线程计划 | 8/10 | 可先用结构化 JSON / DAG 实现 |
| Flash agents 并行执行 | 8.5/10 | 工程成熟，重点是隔离和合并 |
| 能做的线程先做 | 9/10 | DAG 依赖调度即可 |
| 有问题线程先搜索 | 8/10 | 用 blocker + search policy 触发 |
| 搜索后进入次级 Thinker | 7.5/10 | 需要控制升级条件 |
| 父代决策继承 | 7/10 | 继承包不能过长，也不能漏关键约束 |
| 祖 Thinker 节点调频 | 6.5/10 | 最难，需要状态机、观察协议和质量门联动 |
| 多线程结果合并 | 7/10 | 需要冲突检测和 merge policy |
| 整体效率提升 | 8/10 | 适合代码、资料、文档、测试并行 |

最大风险：

- 线程太多，协调成本超过执行收益。
- 子线程继承不足，导致各干各的。
- 子线程继承过多，吞掉速度优势。
- 祖 Thinker 调频太频繁，Pro 调用爆炸。
- 多线程写同一文件或同一决策区域，产生冲突。
- 次级 Thinker 滥用，系统从“快”退化成“到处 Pro”。

---

## 最小落地路线

### Phase 1：静态多线程拆解

- 祖 Thinker 输出 Thread Plan / DAG。
- Flash agents 并行跑可执行线程。
- 所有线程结束后统一合并。

目标：

```text
先证明 Pro 拆解 + Flash 并行确实能提速。
```

### Phase 2：父代继承包

- 每个线程启动时携带 Decision Inheritance Packet。
- 子线程输出必须遵守 return_contract。
- 测试子线程是否跑偏。

目标：

```text
解决并行协作中的方向一致性。
```

### Phase 3：阻塞线程升级

- Flash 遇到不确定先搜索。
- 搜索后仍不明确，进入次级 Thinker。
- 次级 Thinker 输出局部更新包。

目标：

```text
让局部难题不拖停全局。
```

### Phase 4：祖 Thinker checkpoint 调频

- 每个线程节点结束后生成 Thread Observation。
- 祖 Thinker 根据 Observation 做 Tuning Decision。
- 支持 continue / merge / search / escalate / replan / stop / ask_user。

目标：

```text
从一次性 planner 升级为项目总控。
```

### Phase 5：阶段收束与项目停靠站

- 增加 Convergence Strategy。
- 支持手动触发、祖 Thinker 判断触发、phase_end / project_end 触发。
- 收束 Thinker 可以调用受控工具做测试和验证。
- 阶段收束后决定 continue / replan / add_threads / fix_before_continue。
- 总收束后生成交付物、测试结果和最终 Thought Record。

目标：

```text
防止多线程项目局部正确但整体跑偏。
```

### Phase 6：代际切换与新祖 Thinker

- 增加 Generational Handoff。
- 阶段收束后生成阶段总结和项目状态快照。
- 新祖 Thinker 继承阶段结论和决策骨架，不继承完整历史。
- 下一阶段重新生成 Thread Plan。
- 阶段档案进入 Project Memory。

目标：

```text
避免长期项目上下文污染，让项目阶段记录自然形成。
```

### Phase 7：质量评分与自动收束

- 增加 quality_floor。
- 增加 merge_conflict_policy。
- 增加 end-state evaluation。
- 支持自动补线程。
- 支持基于质量分、漂移分、风险分自动触发收束。

目标：

```text
让 Thinker 能判断项目是否真的完成，而不是线程都结束就算完成。
```

---

## 可参考资料与借鉴点

### 1. Anthropic：Building Effective Agents

链接：https://www.anthropic.com/engineering/building-effective-agents

可借鉴点：

- routing：按任务类型路由到不同处理路径。
- parallelization：独立子任务并行执行，提升速度或置信度。
- orchestrator-workers：中央 LLM 动态拆解任务，委托 worker，并综合结果。
- evaluator-optimizer：一个模型生成，另一个模型评价并反馈。

对 agent+++ 的启发：

```text
Thinker 不应做成单一 planner，而应组合并行、编排、评估和反馈循环。
```

尤其是 orchestrator-workers 模式，可以作为祖 Thinker + Flash workers 的外部参考。

### 2. Anthropic：How We Built Our Multi-Agent Research System

链接：https://www.anthropic.com/engineering/multi-agent-research-system

可借鉴点：

- lead agent 规划研究过程。
- lead agent 创建多个 subagents 并行搜索。
- subagents 独立使用工具，返回压缩结果。
- lead agent 综合判断是否继续。
- 生产难点包括状态、错误传播、调试、追踪、异步协调。

对 agent+++ 的启发：

```text
祖 Thinker + Flash 子线程 + Observation 回传 + checkpoint 调频是合理方向。
```

这篇也提醒一个关键风险：

```text
同步等待子 agent 简单但会造成瓶颈；
异步执行更快，但会带来状态一致性和错误传播问题。
```

所以 agent+++ 第一版应采用 checkpoint 调频，而不是完全实时异步调频。

### 3. OpenAI Agents SDK：Agent Orchestration

链接：https://openai.github.io/openai-agents-python/multi_agent/

可借鉴点：

- `Agents as tools`：manager agent 保持控制权，调用 specialist agents 完成有边界的子任务。
- `Handoffs`：triage agent 把任务交给 specialist，specialist 接管后续。
- 代码编排可用 structured output、chain、evaluator loop、parallel execution。

对 agent+++ 的启发：

```text
Flash agent 应更像 bounded tool，而不是接管项目的 handoff agent。
```

祖 Thinker 必须保持最终控制权。

### 4. OpenAI Agents SDK：Handoffs

链接：https://openai.github.io/openai-agents-python/handoffs/

可借鉴点：

- handoff 可以带 input_type。
- handoff 可以有 input_filter。
- handoff 可以动态启用/禁用。
- handoff 可以有 on_handoff 回调。

对 agent+++ 的启发：

```text
次级 Thinker 升级可以借鉴 handoff 的结构，但必须带明确输入过滤和返回契约。
```

### 5. OpenAI Agents SDK：Context Management

链接：https://openai.github.io/openai-agents-python/context/

可借鉴点：

- 区分本地 runtime context 和 LLM 可见上下文。
- 上下文可以通过 instructions、input、tools、retrieval 暴露给模型。
- tool context 可以携带 tool_call_id、tool_name、tool_arguments 等元数据。

对 agent+++ 的启发：

```text
Decision Inheritance Packet 应该是 LLM 可见上下文；
运行状态、预算、线程 ID、权限等应放在 runtime context。
```

不要把所有调度状态都塞进模型上下文。

### 6. AutoGen：SelectorGroupChat

链接：https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/selector-group-chat.html

可借鉴点：

- planning agent 拆任务。
- selector 根据上下文选择下一个 agent。
- 支持角色描述、终止条件、自定义选择函数。

对 agent+++ 的启发：

```text
可以借鉴动态选择 agent 和 termination condition，
但不建议照搬群聊式共享上下文。
```

agent+++ 更应该由祖 Thinker 分发继承包，子线程局部执行，再结构化回传。

### 7. TDAG：Dynamic Task Decomposition and Agent Generation

链接：https://arxiv.org/abs/2402.10178

可借鉴点：

- 动态任务分解。
- 为子任务生成专门 subagent。
- 强调复杂任务中的适应性和上下文意识。

对 agent+++ 的启发：

```text
Thread Plan 应支持运行中 replan，而不是一次性固定计划。
```

### 8. AgentOrchestra

链接：https://arxiv.org/abs/2506.12508

可借鉴点：

- 层级多 agent 框架。
- 中央 planning agent 做高层规划。
- 模块化 agent 协作。
- 显式 sub-goal formulation 和 adaptive role allocation。

对 agent+++ 的启发：

```text
Thinker 可以采用层级结构，但要把父代决策继承做成显式协议。
```

### 9. AdaptOrch

链接：https://arxiv.org/abs/2602.16873

可借鉴点：

- 根据任务依赖图选择 parallel / sequential / hierarchical / hybrid 拓扑。
- 把 orchestration topology 当作系统性能优化目标。
- 强调拓扑选择可能比单模型选择更重要。

对 agent+++ 的启发：

```text
Thinker 的核心不是“用哪个模型”，而是“这个任务该用什么协作拓扑”。
```

### 10. Agent Capsules

链接：https://arxiv.org/abs/2605.00410

可借鉴点：

- 用质量门控制 multi-agent pipeline 的执行粒度。
- 不盲目拆分或合并 agent。
- 每次模式切换都受 output quality 约束。

对 agent+++ 的启发：

```text
Thinker 需要线程预算、质量门和升级阈值，避免多 agent 失控。
```

### 11. AgentGit

链接：https://arxiv.org/abs/2511.00628

可借鉴点：

- 给多 agent 工作流加 commit / revert / branching。
- 支持并行探索、回滚、轨迹比较。

对 agent+++ 的启发：

```text
多线程执行尤其是代码修改，需要 branch / checkpoint / rollback 思维。
```

这可以用于后续解决多 Flash agents 并行改文件的冲突问题。

---

## 和现有参考的差异

公开资料可以支撑三个判断：

```text
1. 层级 orchestrator-worker 是有效主流模式。
2. 并行 subagents 对广度探索和复杂搜索有明显价值。
3. 生产级难点集中在协调、状态、评估、调试、成本控制。
```

但 agent+++ Thinker 的差异点是：

```text
1. 父代决策继承
2. 祖 Thinker 节点调频
3. Observation Contract 作为线程反馈协议
4. Global Precipitation Memory 作为 Thought Record 的长期沉淀层
```

也就是说，agent+++ 不是简单复刻 multi-agent orchestration。

它要做的是：

```text
带父代决策血统和祖级调频的项目决策拆解系统。
```

---

## 可分享版总结

Thinker 是 agent+++ 的项目决策拆解大师。

它让 Pro 模型不再陷入工具执行和日志阅读，而是专注于高价值判断：拆解项目、分配线程、传递决策边界、调频子任务、合并结果和沉淀思考。

普通 Agent 的瓶颈是：

```text
单线程思考 + 单线程执行
```

agent+++ Thinker 的目标是：

```text
祖 Thinker 深度决策
Flash agents 并行执行
次级 Thinker 局部补洞
Observation Contract 结构化回传
祖 Thinker checkpoint 调频
收束 Thinker 阶段停靠和项目验收
Generational Handoff 生成新祖 Thinker
```

这套结构的价值不是“多几个 agent”，而是让深度和速度不再互相牺牲。
