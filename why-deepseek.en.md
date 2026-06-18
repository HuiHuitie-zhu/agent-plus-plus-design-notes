# Why DeepSeek as the Model Base for agent+++

> Date: 2026-06-19  
> Status: public draft  
> Related modules: Thinker / Reflex Layer / Observation Contract / Global Precipitation Memory  
> Purpose: explain why agent+++ uses DeepSeek as the default model base and how far DeepSeek is expected to support the system.

---

## One-Sentence Answer

**agent+++ uses DeepSeek not because the architecture should be locked to one provider, but because DeepSeek's Flash / Pro split, reasoning capacity, cost structure, and long-context capabilities fit a project operating system built around deep Thinker decisions, high-frequency Flash execution, and high token utilization.**

The model strategy is not:

```text
use one strongest model for everything
```

It is:

```text
Pro for deep project decisions
Flash for high-frequency execution and feedback handling
Observation Contract for higher token quality
Generational Handoff for long-project context hygiene
```

---

## Why Not Just Use the Strongest Model Everywhere

A long-running project agent is not a single-turn chat product.

It continuously performs:

```text
requirement clarification
thread decomposition
tool invocation
feedback parsing
search and evidence gathering
local escalation
convergence checkpointing
memory extraction
next-stage continuation
```

These steps create many model calls. If every step uses the most expensive and deepest model, a multi-threaded agent quickly becomes impractical.

agent+++'s position is:

> Model capability should be placed into system roles, not used uniformly.

---

## DeepSeek Roles in agent+++

### DeepSeek Pro: Deep Judgment

DeepSeek Pro is used for high-value Thinker nodes:

- ancestor Thinker project decisions
- complex goal decomposition
- parent decision chain generation
- secondary Thinker for local hard problems
- convergence checkpointing
- generational handoff
- important architecture decisions
- high-risk trade-offs

These calls should be lower frequency but high impact.

```text
low frequency
high value
deep reasoning
project-direction impact
```

### DeepSeek Flash: High-Frequency Execution

DeepSeek Flash is used for Reflex Layer and feedback handling:

- thread-level small tasks
- tool-output normalization
- local file understanding
- low-risk summarization
- initial Observation Contract classification
- initial memory candidate filtering
- user-facing concise feedback

These calls are high-volume, lower-risk, and often verifiable.

```text
high frequency
lower cost
low latency
verifiable
```

---

## Why DeepSeek Fits Multi-Threaded Agents

agent+++'s Thinker architecture depends on multi-thread collaboration:

```text
Ancestor Thinker
  -> Thread Plan
  -> parallel Flash threads
  -> blocked threads search or escalate
  -> Observation Contract return
  -> ancestor tuning
  -> Convergence Dock
```

This requires two properties:

1. Deep nodes must reason well.
2. High-frequency nodes must be affordable and fast enough.

DeepSeek's Pro / Flash split fits this structure.

If the system only has a strong model, it becomes slow and expensive.  
If it only has a fast model, it lacks deep project judgment.

agent+++ needs both:

```text
deep decisions on Pro
high-frequency execution on Flash
```

---

## What DeepSeek Supports in agent+++

### 1. Basic Conversation and Streaming

Used for:

- normal user communication
- project clarification
- phase summaries
- user-readable feedback

### 2. Structured Thinker Output

Thinker needs stable structured outputs:

```text
ThreadPlan
DecisionInheritancePacket
ThreadObservation
TuningDecision
ConvergenceStrategy
GenerationalHandoff
ThoughtRecord
```

Even without relying only on strict native function calling, the system can use:

```text
structured prompts
JSON schema validation
repair loops
Observation Contract re-labeling
```

to keep outputs controllable.

### 3. Tool-Orchestration Participation

agent+++ does not give tools directly and freely to the model.

The safer loop is:

```text
model outputs execution intent
system performs permission and safety checks
Reflex Layer invokes tools
Observation Contract returns structured feedback
Thinker decides the next step
```

DeepSeek participates in the full agent loop without becoming an unbounded tool executor.

### 4. Long Context and Context Budgeting

DeepSeek's long-context capability is useful for long projects, but agent+++ does not treat long context as an unbounded container for raw material.

Long context should carry:

```text
structured project state
stage archives
evidence summaries
current decision chain
output and tuning reserve
```

It should not carry every log, conversation, and file in full.

---

## High Token Utilization

agent+++ does not simply try to "save tokens." It tries to spend tokens where they have the highest decision value.

### 1. Raw Logs Do Not Go Directly to Thinker

Tool output passes through Observation Contract:

```text
raw output
  -> facts / risks / artifacts / evidence_refs
  -> summary_for_thinker
```

Thinker reads decision material, not execution noise.

### 2. Child Threads Inherit Decision Skeletons

Each Flash thread receives:

```text
parent goal
decision frame
constraints
quality bar
thread scope
done condition
escalation triggers
```

This is the Decision Inheritance Packet.

It is cheaper and safer than injecting the full project history into every child agent.

### 3. Project Memory Is Projected by Relevance

Project Memory is not a full historical dump. It is projected from global precipitation:

```text
Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
  -> Thinker Context
```

### 4. Long Projects Use Generational Handoff

Long projects should not survive through endless context compression:

```text
old Ancestor Thinker
  -> Convergence Dock
  -> Generational Handoff
  -> new Ancestor Thinker
```

The new ancestor inherits stage conclusions and boundaries, not the full trace.

In short:

```text
compression makes old material smaller;
generational handoff inherits only conclusions and boundaries
```

### 5. High-Fidelity Evidence Opens on Demand

Observation Contract uses fidelity policies:

```text
drop
summary
extract
excerpt
raw_window
full_raw
```

Thinker does not receive full logs by default. Raw evidence opens only when the user requests it, verification needs it, or a key decision requires it.

---

## DeepSeek Is Not a Lock-In

DeepSeek is the current default model base, but the architecture should not be tied to a single provider.

The system should abstract:

```text
Thinker Model
Reflex Model
Observation Model
Memory Extraction Model
```

Another model can replace a role later, while the role boundary remains.

The key point:

> The competitive advantage is not which model is connected, but what project operating system the model is placed inside.

---

## Risks and Boundaries

### 1. Pro Must Not Be Overused

If every blocker upgrades to Pro, the multi-threaded system becomes slow and expensive.

The system needs:

```text
max_secondary_thinkers
max_search_rounds_per_thread
max_replans
max_convergence_rounds
```

### 2. Flash Must Not Rewrite Parent Decisions

Flash can execute, report, and suggest. It cannot change the ancestor Thinker's decision frame on its own.

### 3. Long Context Does Not Replace Memory Governance

Even with long context, the system still needs:

```text
Observation Contract
Global Precipitation Memory
Project Memory View
Generational Handoff
```

Otherwise context pollution is merely delayed.

---

## Summary

agent+++ chooses DeepSeek because it fits the system's operating pattern:

```text
Pro for deep judgment
Flash for high-frequency execution
Observation Contract for better feedback quality
Global Precipitation Memory for long-term memory control
Generational Handoff for long-project context hygiene
```

DeepSeek is not just a provider string in agent+++. It is the default model base.

But the deeper claim is not that one model is universally best. The claim is:

> Agents become powerful when models are placed inside the right architecture for thinking, execution, feedback, and memory.
