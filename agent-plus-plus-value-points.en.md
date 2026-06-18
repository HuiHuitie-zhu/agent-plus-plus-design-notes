# agent+++ Value Points

> Date: 2026-06-18  
> Status: public draft  
> Purpose: a concise English overview of the core value of agent+++ for product, engineering, open-source, and partner conversations.  
> Core version: four-module architecture.

---

## One-Sentence Positioning

**agent+++ is a local-first project-oriented agent workspace that turns fragmented human ideas into executable, verifiable, and reusable project assets through four core modules: Thinker, Reflex Layer, Feedback Bus, and Project Memory.**

It is not an "open-source Codex clone" and it is not a low-code generator. The core idea is to build a reliable human-AI project operating system:

```text
human: imagination, judgment, acceptance
system: clarification, execution, feedback, memory
```

---

## Core Thesis

Most agent products solve two problems:

1. **Execution**: the user already knows what to do, and the agent writes code, runs commands, or searches.
2. **Conversation**: the user discusses, brainstorms, or generates content with AI.

Real work has a third and harder problem:

> Many projects do not start with a complete requirement. They start with a sentence, a link, a fragment, a failed attempt, or a half-formed idea.

agent+++ tries to systematize this chain:

```text
vague idea
  -> coaching clarification
  -> product contract
  -> technical decomposition
  -> execution and verification
  -> structured feedback
  -> project memory
  -> continuation
```

The value is not any single feature. The value is the clear boundary and cooperation between the four modules.

---

## Four Core Modules

### 1. Thinker

**Responsibility: intent judgment, contract generation, project decision-making, multi-thread decomposition, tuning, convergence, and generational handoff.**

Thinker corresponds to Pro-level reasoning. It handles high-value judgment:

- Interprets user intent and project stage.
- Turns vague ideas into Product Contracts.
- Decomposes work into Thread Plans.
- Sends bounded work to Flash execution agents.
- Reads structured feedback instead of raw logs.
- Performs checkpoint tuning after thread observations.
- Runs convergence docks for phase or project-level validation.
- Generates generational handoff so long projects continue with clean context.

Thinker is not a one-shot planner. It is a project decision system.

```text
Ancestor Thinker
  -> Thread Plan
  -> Decision Inheritance Packet
  -> Flash parallel execution
  -> Thread Observation
  -> Tuning Decision
  -> Convergence Dock
  -> Generational Handoff
```

### 2. Reflex Layer

**Responsibility: fast execution, tool invocation, file operations, search, and raw result collection.**

Reflex Layer corresponds to Flash models and tool executors. It is the system's hands and eyes:

- Runs shell commands.
- Reads and writes files.
- Calls local tools and external capabilities.
- Executes thread-level tasks in parallel where safe.
- Sends raw results to Feedback Bus.

Its principle:

```text
execute fast, do not own high-level decisions
```

This keeps execution cheap and fast while preventing local noise from contaminating project-level reasoning.

### 3. Feedback Bus / Observation Contract

**Responsibility: turn raw tool outputs into structured observations that Thinker can reason over.**

Most agents put raw tool output back into the model context. That works for short tasks, but long tasks quickly suffer from:

- **Context pollution**: logs, file lists, diffs, and traceback noise drown out the facts.
- **Context distortion**: crude summaries remove the evidence needed for next-step decisions.

agent+++ uses Observation Contract as a multi-axis separator:

```text
provenance      where the information comes from
determinism     whether the output is stable
epistemic       evidence strength
semantic_role   role in the task
actionability   whether it should affect the next step
fidelity        how much raw evidence to preserve
```

The point is not token reduction alone. The point is to reconstruct observations:

```text
raw tool output
  -> evidence judgment
  -> semantic role
  -> action value
  -> fidelity policy
  -> route to Thinker / Memory / UI / Audit / Verifier
```

### 4. Project Memory / Global Precipitation Memory

**Responsibility: turn valuable observations and decisions into reusable project context.**

Project Memory is not the first storage layer. The first precipitation layer should be global:

```text
Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

The global layer temporarily holds unassigned observations, Pro thought records, user cognition, and cross-project lessons. Project Memory is a project-scoped projection extracted from that global layer.

Unassigned memory does not grow forever. It goes through a lifecycle:

```text
Retention
  -> Hallucination
  -> Oblivion
```

Retention waits for project relevance. Hallucination turns long-unclaimed fragments into creative residue for creative tasks only. Oblivion permanently deletes low-value fragments.

---

## Why This Is Different

普通 agent loop:

```text
model thinks
  -> tool call
  -> raw output back to model
  -> repeat
```

agent+++ loop:

```text
Thinker decides
  -> Reflex executes
  -> Observation Contract structures feedback
  -> Thinker tunes or converges
  -> Memory precipitates
  -> next generation continues
```

This is a shift from task execution to project operation.

---

## DeepSeek Model Strategy

agent+++ is designed around model role separation:

```text
DeepSeek Flash:
  - high-frequency execution
  - local thread tasks
  - tool-output normalization
  - low-risk summaries
  - initial memory candidate filtering

DeepSeek Pro:
  - ancestor Thinker decisions
  - secondary Thinker for local hard problems
  - convergence checkpoints
  - generational handoff
  - important architecture decisions
```

The goal is high token utilization:

- Raw logs do not go directly to Thinker.
- Child threads inherit decision skeletons instead of full context.
- Project Memory is projected from global precipitation, not dumped wholesale.
- Long projects use generational handoff instead of endless context compression.
- High-fidelity evidence opens only when needed.

---

## Competitive Value

agent+++ is not just "more agents." Its value is the operating system layer around agents:

- Observation Contract prevents feedback distortion.
- Decision Inheritance Packet prevents child threads from drifting.
- Tuning Decision keeps parallel work aligned.
- Convergence Dock prevents projects from locally succeeding but globally drifting.
- Generational Handoff keeps long projects clean and fast.
- Global Precipitation Memory prevents memory pollution and gives unclaimed fragments a lifecycle.

If completed well, agent+++ can become:

> a local-first AI project operating system for people and small teams who need long-running, evidence-aware, memory-aware project collaboration.

---

## Current Status

This is a public design note, not a claim that every feature is complete.

Current state:

```text
Core architecture: formed
Observation Contract: engineering prototype exists
Thinker: architecture finalized, implementation pending
Global Precipitation Memory: architecture formed, implementation pending
Project Memory: needs to evolve into project projection
Reflex Layer: basic execution exists, multi-thread scheduling pending
UI: needs thread, tuning, convergence, and stage-record visibility
```

The documents are intended to share the design direction and invite discussion.
