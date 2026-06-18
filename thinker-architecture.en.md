# Thinker Architecture: Project Decision Decomposition Master

> Date: 2026-06-18  
> Status: practical design draft  
> Module: Thinker  
> Related modules: Observation Contract / Feedback Bus / Reflex Layer / Global Precipitation Memory / Project Memory  
> Purpose: document the confirmed Thinker architecture, key protocols, implementation path, and external references.

---

## One-Sentence Definition

**Thinker is agent+++'s project decision decomposition master: it uses a Pro model for deep judgment, decomposes complex projects into parallel executable threads, and continuously tunes child execution so speed, quality, and direction stay aligned.**

Thinker is not a normal planner.

It is not a todo-list generator and not a single Pro call. Its core chain is:

```text
deep decision
  -> multi-thread decomposition
  -> parent decision inheritance
  -> parallel Flash execution
  -> local escalation
  -> ancestor Thinker tuning
  -> convergence and merge
  -> generational handoff
```

This lets agent+++ do:

```text
Pro for deepest judgment
Flash for maximum throughput
secondary Thinker for local hard problems
ancestor Thinker for global quality and direction
```

---

## Architecture Position

```text
User Intent
  ↓
Thinker
  ↓
Reflex Layer
  ↓
Feedback Bus / Observation Contract
  ↓
Thinker tuning
  ↓
Global Precipitation Memory / Project Memory
```

Thinker consumes structured context, not raw logs:

```text
Project Memory View
Projected Global Memory
Observation Contract
Execution Feedback
Open Questions
User Intent
```

It outputs structured decisions:

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

## Core Responsibilities

### 1. Decision Decomposition

Ancestor Thinker reads the user goal, project memory, and recent observations, then decomposes a complex task into executable threads.

The output is not a linear todo list. It is a dependency-aware Thread Plan / DAG.

### 2. Multi-Thread Coordination

Threads that can run immediately are assigned to Flash agents in parallel.

Blocked threads do not stop the whole project:

```text
runnable threads continue
blocked thread searches once
if still unclear, escalate to secondary Thinker
secondary Thinker resolves local uncertainty
Flash continues execution
```

### 3. Parent Decision Inheritance

Each child thread must inherit the parent's decision skeleton before it starts.

The child does not inherit full project history. It inherits:

```text
parent goal
decision frame
accepted assumptions
rejected options
quality bar
constraints
thread scope
escalation triggers
```

### 4. Ancestor-Level Tuning

Ancestor Thinker does not disappear after dispatching tasks.

After each child node finishes, it returns a structured observation through Observation Contract. Ancestor Thinker then decides:

```text
continue
merge
search
escalate to secondary Thinker
replan
stop
ask user
```

This is the quality-control core.

### 5. Convergence Dock

Multi-thread systems need docking stations.

At manual triggers, ancestor judgment, phase end, or project end, Thinker enters a convergence stage:

```text
collect thread results
organize artifacts
merge project state
run controlled tests
check drift
decide continue / replan / finish
```

Convergence Thinker is allowed to use tools, but only through an explicit Convergence Strategy.

### 6. Generational Handoff

After convergence, if the project continues, the original ancestor Thinker should not carry all historical context forever.

Instead:

```text
old Ancestor Thinker
  -> Convergence Dock
  -> phase summary / project snapshot / new decision chain
  -> new Ancestor Thinker
  -> next-stage Thread Plan
```

Principle:

```text
new ancestor inherits stage conclusions and boundaries,
not the full historical trace
```

This keeps long projects clean, fast, and well documented.

---

## Main Runtime Chain

```text
1. Project Context Compose
   Build context from Project Memory View, projected global memory, user input, and recent observations.

2. Ancestor Thinker Reasoning
   Pro reasoning determines goal, constraints, quality bar, and decomposition strategy.

3. Decision Decomposition
   Produce Thread Plan / DAG.

4. Inheritance Build
   Create Decision Inheritance Packet for each thread.

5. Parallel Flash Execution
   Runnable threads execute in parallel.

6. Thread Observation
   Each thread returns structured Observation Contract output.

7. Tuning Checkpoint
   Ancestor Thinker tunes based on thread observations.

8. Local Escalation
   Blocked threads may search, then escalate to secondary Thinker.

9. Merge & Replan
   Merge results, detect conflicts, and replan if needed.

10. Convergence Dock
    Organize outputs, merge state, run controlled tests, and check drift.

11. Generational Handoff
    If the project continues, create a new decision chain for the next ancestor Thinker.

12. Thought Record
    Produce a structured reasoning record for Observation Contract and global precipitation.
```

---

## Protocol 1: Thread Plan

Thread Plan is a DAG, not a sequential todo list.

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

It answers:

```text
What is this thread responsible for?
What does it depend on?
Can it start now?
Who should execute it?
What counts as done?
When should it escalate?
How should its result merge?
```

---

## Protocol 2: Decision Inheritance Packet

Decision Inheritance Packet is the parent-decision inheritance protocol.

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

Principle:

```text
children inherit decision boundaries, not full parent context
```

This prevents drift without destroying speed.

---

## Protocol 3: Thread Observation

Thread completion must not return only `done` or `failed`.

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

Observation Contract labels it with provenance, epistemic grade, semantic role, actionability, and fidelity.

---

## Protocol 4: Tuning Decision

Ancestor Thinker makes a tuning decision at each checkpoint:

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

It checks:

```text
Does this follow the parent plan?
Did new risk appear?
Is more evidence needed?
Should this local issue become a secondary Thinker task?
Can this result merge into the main line?
Does it conflict with other threads?
Does it require user confirmation?
```

---

## Protocol 5: Convergence Strategy

Convergence Strategy handles phase-level and project-level docking.

```python
class ConvergenceStrategy:
    convergence_id: str
    parent_thinker_id: str
    trigger: str
    # manual / ancestor_judged / phase_end / project_end / risk_threshold / drift_detected

    scope: str
    # phase / project / thread_group

    inputs: list[str]
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

Triggers:

```text
manual request
ancestor judgment
phase end
project end
risk threshold
drift detected
```

Convergence is not a summary. It is project docking and validation.

---

## Protocol 6: Generational Handoff

Generational Handoff happens after convergence when the project continues.

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

It outputs:

1. **Stage archive** for Project Memory:

```text
stage goal
stage outputs
validation results
key decisions
abandoned paths
remaining risks
next entry point
```

2. **New parent decision skeleton** for the next ancestor:

```text
next_phase_goal
next_decision_frame
inherited_constraints
next_quality_bar
recommended_thread_plan
```

Compression makes old material smaller. Generational handoff inherits only conclusions and boundaries.

---

## State Machines

Thread state:

```text
planned
  -> ready
  -> running_flash
  -> observed
  -> checkpoint_review
  -> continue / merge / search / escalate / replan / ask_user / stop
  -> convergence_dock
  -> validated / replan / finish
```

Project state:

```text
planning
  -> parallel_execution
  -> checkpoint_tuning
  -> phase_convergence
  -> generational_handoff
  -> new_ancestor_thinker
  -> next_phase or replan
  -> final_convergence
  -> user_review / finish
```

---

## Collaboration Rules

### 1. Flash agents cannot rewrite parent decisions

Flash agents may:

- execute
- collect facts
- produce artifacts
- report risks
- suggest next steps

Flash agents may not:

- change parent goals
- expand scope on their own
- overturn the ancestor decision frame
- upgrade local findings into global decisions

### 2. Secondary Thinker handles local hard problems only

It solves uncertainty inside a thread. It does not take over the whole project.

### 3. Every thread needs a done condition

No done condition, no thread launch.

### 4. Every merge preserves evidence

Merge must explain:

```text
which thread contributed what
whether there were conflicts
which result was chosen
why
where the evidence is
```

### 5. Multi-thread projects need convergence points

Local success can still produce global drift. Each major phase needs a convergence checkpoint.

### 6. Convergence Thinker can use tools, but only under control

Allowed tools, test commands, validation tasks, and acceptance criteria must be declared in Convergence Strategy.

### 7. Long projects need generational handoff

Do not let one ancestor Thinker live forever. After important phase convergence:

```text
old ancestor exits
stage archive enters Project Memory
new ancestor inherits clean decision skeleton
next stage gets a new Thread Plan
```

---

## Budgets and Quality Gates

Thinker must have explicit budgets:

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

Suggested defaults:

```text
normal task: 2-4 Flash threads
complex project: 4-8 Flash threads
search: 1-2 rounds per thread
secondary Thinker: 1-2 by default
tuning: checkpoint after node completion, not high-frequency real-time tuning
phase convergence: at least once per major phase
final convergence: always before project completion
generational handoff: default after phase convergence in long projects
```

---

## Implementation Path

### Phase 1: Static Multi-Thread Decomposition

- Ancestor Thinker outputs Thread Plan / DAG.
- Flash agents execute runnable threads in parallel.
- Merge at the end.

### Phase 2: Decision Inheritance Packet

- Each thread starts with a parent decision skeleton.
- Thread outputs must follow return contracts.
- Test for drift.

### Phase 3: Blocked Thread Escalation

- Flash searches once when blocked.
- If still unclear, escalate to secondary Thinker.
- Secondary Thinker returns local update.

### Phase 4: Ancestor Checkpoint Tuning

- Thread Observation after each node.
- Ancestor Thinker makes Tuning Decision.
- Support continue / merge / search / escalate / replan / stop / ask_user.

### Phase 5: Convergence Dock

- Add Convergence Strategy.
- Support manual, ancestor-judged, phase-end, and project-end triggers.
- Let Convergence Thinker run controlled tests and validation.

### Phase 6: Generational Handoff

- Generate stage summary and project snapshot after convergence.
- New ancestor inherits conclusions and decision skeleton, not full history.
- Stage archive enters Project Memory.

### Phase 7: Quality Scoring and Automatic Convergence

- Add quality floor.
- Add merge conflict policy.
- Add end-state evaluation.
- Trigger convergence based on quality, drift, and risk.

---

## Feasibility

Overall feasibility: **7.5/10**.

| Module | Feasibility | Notes |
|---|---:|---|
| Ancestor multi-thread planning | 8/10 | Start with structured JSON / DAG |
| Flash parallel execution | 8.5/10 | Mature engineering, needs isolation and merge |
| Runnable threads first | 9/10 | DAG scheduling |
| Blocked thread search | 8/10 | Trigger by blocker and search policy |
| Secondary Thinker escalation | 7.5/10 | Must control escalation |
| Parent decision inheritance | 7/10 | Packet must be short but complete |
| Ancestor tuning | 6.5/10 | Hardest part |
| Multi-thread merge | 7/10 | Needs conflict detection |
| Overall speedup | 8/10 | Especially useful for code, docs, research, tests |

---

## External References

### Anthropic: Building Effective Agents

https://www.anthropic.com/engineering/building-effective-agents

Relevant ideas:

- routing
- parallelization
- orchestrator-workers
- evaluator-optimizer

Takeaway:

```text
Thinker should combine orchestration, parallelism, and feedback loops rather than act as a single planner.
```

### Anthropic: Multi-Agent Research System

https://www.anthropic.com/engineering/multi-agent-research-system

Relevant ideas:

- lead agent plans
- subagents search in parallel
- subagents return compressed findings
- lead agent synthesizes and decides whether to continue

Takeaway:

```text
Ancestor Thinker + Flash threads + Observation return + checkpoint tuning is a grounded architecture.
```

### OpenAI Agents SDK: Multi-Agent Orchestration

https://openai.github.io/openai-agents-python/multi_agent/

Relevant ideas:

- agents as tools
- handoffs
- structured output
- evaluator loops
- parallel execution

Takeaway:

```text
Flash agents should behave like bounded tools. Ancestor Thinker should retain control.
```

### AutoGen: SelectorGroupChat

https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/selector-group-chat.html

Relevant ideas:

- planning agent
- dynamic agent selection
- termination conditions

Takeaway:

```text
Dynamic selection is useful, but agent+++ should avoid full shared group-chat context.
```

### TDAG / AgentOrchestra / AdaptOrch / Agent Capsules / AgentGit

These research directions support:

- dynamic task decomposition
- hierarchical agents
- topology-aware orchestration
- quality gates
- branch / checkpoint / rollback for multi-agent workflows

agent+++'s difference is to make these ideas project-operational through:

```text
Decision Inheritance Packet
Tuning Decision
Observation Contract
Convergence Strategy
Generational Handoff
```

---

## Summary

Thinker is agent+++'s project decision decomposition master.

It keeps Pro models focused on judgment, not logs or low-level execution. It turns deep reasoning into parallel Flash execution, tunes based on structured observations, docks at phase boundaries, and hands off clean decision chains to the next project generation.

Ordinary agents are limited by:

```text
single-thread thinking + single-thread execution
```

agent+++ aims for:

```text
ancestor Thinker deep decision
parallel Flash execution
secondary Thinker local repair
Observation Contract structured feedback
Convergence Dock validation
Generational Handoff continuation
```

The value is not "more agents." The value is an operating system for making multi-agent work controllable, fast, and project-aware.
