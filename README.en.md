# agent+++ Design Notes

> English version  
> Status: public draft  
> Date: 2026-06-18

agent+++ is a design for a local-first agent workspace built for long-running projects.

It is not just a chatbot, and it is not merely a coding agent that runs commands. The project focuses on a harder question:

> How can an AI system keep moving a real project forward, instead of only completing isolated tasks?

These design notes describe the current architecture of agent+++: how Thinker, Reflex Layer, Feedback Bus, Observation Contract, Global Precipitation Memory, and Project Memory work together to turn fragmented human ideas into executable, verifiable, and reusable project assets.

---

## One-Sentence Positioning

**agent+++ is a local-first project-oriented agent operating system: a deep Thinker decomposes decisions, Flash execution agents move work forward in parallel, Observation Contract keeps feedback faithful, and global/project memory lets long-running projects continue cleanly across stages.**

Most agents look like this:

```text
user task -> agent execution -> result
```

agent+++ aims for this:

```text
vague idea
  -> clarification
  -> decision decomposition
  -> parallel execution
  -> structured observation
  -> convergence checkpoint
  -> memory precipitation
  -> next project stage
```

---

## Public Documents

Suggested reading order:

1. [agent-plus-plus-value-points.md](agent-plus-plus-value-points.md)  
   Project overview: four core modules, value proposition, and product positioning.
   English: [agent-plus-plus-value-points.en.md](agent-plus-plus-value-points.en.md)

2. [observation-contract.md](observation-contract.md)  
   Observation Contract: turning raw tool outputs into reasoning-ready, auditable, memory-worthy observation assets.
   English: [observation-contract.en.md](observation-contract.en.md)

3. [global-precipitation-memory.md](global-precipitation-memory.md)  
   Global Precipitation Memory: global staging, project memory projection, retention, hallucination, and forgetting lifecycle.
   English: [global-precipitation-memory.en.md](global-precipitation-memory.en.md)

4. [thinker-architecture.md](thinker-architecture.md)  
   Thinker Architecture: project decision decomposition, multi-thread tuning, convergence dock, and generational handoff.
   English: [thinker-architecture.en.md](thinker-architecture.en.md)

---

## Four Core Modules

### 1. Thinker

Thinker is the project decision decomposition master.

It uses a Pro model for high-value judgment, decomposes complex projects into parallelizable threads, and continuously tunes execution:

```text
Ancestor Thinker
  -> Thread Plan
  -> Decision Inheritance Packet
  -> parallel Flash execution
  -> Thread Observation
  -> Tuning Decision
  -> Convergence Dock
  -> Generational Handoff
```

The goal is not to let a Pro model slowly do everything. The goal is to let Pro handle the deepest decisions, while Flash agents provide maximum execution throughput.

### 2. Reflex Layer

Reflex Layer is the fast execution layer.

It reads and writes files, invokes tools, runs commands, searches, and completes small execution steps. It does not make high-level project decisions. It follows Thread Plans and Decision Inheritance Packets produced by Thinker.

### 3. Feedback Bus / Observation Contract

Feedback Bus is not a log compressor.

Observation Contract separates tool outputs into multiple dimensions:

```text
provenance
determinism
epistemic
semantic_role
actionability
fidelity
```

This lets the system distinguish:

```text
What is a fact?
What is only model interpretation?
What blocks the next step?
What needs raw evidence?
What deserves to enter memory?
```

### 4. Project Memory / Global Precipitation Memory

Project Memory is not the first storage layer.

The first precipitation layer should be global:

```text
Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

The global layer temporarily holds unassigned observations, Pro thought records, and user cognition. Project Memory is the project-scoped view projected from that global layer.

---

## Why DeepSeek as the Model Base

agent+++ does not rely on a single strongest model for everything.

Its model strategy is:

```text
DeepSeek Flash for high-frequency execution and feedback handling
DeepSeek Pro for deep reasoning and project-level decisions
```

DeepSeek is a good default base for four reasons.

### 1. Cost Structure Fits Multi-Threaded Agents

agent+++ is not a single-turn Q&A system. It performs many small steps during long-running projects:

```text
thread decomposition
tool invocation
feedback parsing
convergence checkpoints
memory extraction
local escalation
```

If every step used an expensive frontier model, the economics of a multi-threaded system would collapse quickly.

The Flash / Pro split fits the architecture:

```text
high-frequency, low-risk, verifiable actions -> Flash
low-frequency, high-value strategic decisions -> Pro
```

### 2. Reasoning Capacity Fits Thinker

Thinker needs more than chat ability. It needs to:

- decompose complex goals
- preserve constraints
- identify risks
- generate decision chains
- retune based on feedback
- perform convergence and generational handoff

DeepSeek Pro is positioned as the model used for these high-value reasoning nodes.

### 3. Flash Fits Reflex Layer

Reflex Layer needs speed, low cost, and high-volume repetition:

- reading files
- summarizing local outputs
- parsing tool results
- executing thread-level tasks
- producing structured returns

These steps do not need the strongest model every time. Flash plus Observation Contract can turn large volumes of execution feedback into high-quality observations for Thinker.

### 4. Good Engineering Fit for a Local-First Workspace

agent+++ aims to be local-first, controllable, and auditable.

DeepSeek can be wrapped behind replaceable model backends:

```text
Thinker Model
Reflex Model
Observation Model
Memory Extraction Model
```

DeepSeek is the current default base, but the architecture is not locked to one model provider.

---

## What DeepSeek Supports in agent+++

DeepSeek support in agent+++ can be understood in four layers.

### 1. Basic Conversation and Streaming

Used for normal conversation, project clarification, phase summaries, and user-facing communication.

### 2. Structured Decision Output

Thinker needs to produce structured objects:

```text
ThreadPlan
DecisionInheritancePacket
TuningDecision
ConvergenceStrategy
GenerationalHandoff
ThoughtRecord
```

Even when a model is not used through strict native function calling, agent+++ can rely on structured prompts, JSON schema validation, and repair loops to obtain stable outputs.

### 3. Tool-Orchestration Participation

agent+++ does not require the model to freely call every tool.

The safer loop is:

```text
model outputs execution intent
system performs permission and safety checks
Reflex Layer invokes tools
Observation Contract returns structured feedback
Thinker decides the next step
```

This allows DeepSeek to participate in the full agent loop without giving the model unbounded tool authority.

### 4. Multi-Model Role Separation

DeepSeek Flash and Pro serve different roles:

```text
Flash:
  - tool-result normalization
  - local thread execution
  - low-risk summarization
  - initial memory candidate filtering

Pro:
  - ancestor Thinker decisions
  - secondary Thinker for local hard problems
  - convergence checkpoints
  - generational handoff
  - important architecture decisions
```

This is not simply mixing small and large models. It is assigning model capability to explicit system roles.

---

## High Token Utilization

agent+++ does not treat token reduction as the final goal. It tries to spend tokens where they matter most.

Core strategies:

### 1. Raw Logs Do Not Go Directly to Thinker

Tool outputs pass through Observation Contract first:

```text
raw output
  -> facts / risks / artifacts / evidence_refs
  -> summary_for_thinker
```

Thinker reads decision material, not noisy execution transcripts.

### 2. Child Threads Inherit Decision Skeletons, Not Full Context

Each Flash thread receives a Decision Inheritance Packet:

```text
parent goal
constraints
quality bar
thread scope
done condition
escalation triggers
```

This is cheaper and safer than injecting the full project history into every sub-agent.

### 3. Project Memory Comes from Relevance Projection

Project Memory does not dump all history into context. It is projected from the global precipitation layer based on the current project and task:

```text
Global Precipitation Layer
  -> relevant memory projection
  -> Project Memory View
  -> Thinker Context
```

### 4. Long Projects Use Generational Handoff

Long-running projects should not survive by endlessly compressing old context.

Instead:

```text
old Ancestor Thinker
  -> convergence checkpoint
  -> Generational Handoff
  -> new Ancestor Thinker
```

The new ancestor inherits stage conclusions and boundaries, not the full historical trace.

In short:

```text
compression makes old material smaller;
generational handoff inherits only conclusions and boundaries.
```

### 5. High-Fidelity Evidence Opens on Demand

Observation Contract supports a Fidelity Policy:

```text
drop / summary / extract / excerpt / raw_window / full_raw
```

Thinker does not receive full logs by default. Raw evidence is opened only when the user asks, verification requires it, or a key decision needs high-fidelity support.

---

## Current Status

These documents are public design notes for agent+++. They do not claim that every feature has already been fully productized.

Current status:

```text
Core architecture: formed
Observation Contract: engineering prototype exists
Thinker: architecture finalized, implementation pending
Global Precipitation Memory: architecture formed, implementation pending
Project Memory: needs to evolve into a projection from global precipitation
Reflex Layer: basic execution exists, multi-thread scheduling needs work
UI: needs thread, tuning, convergence, and stage-record visibility
```

The value of these notes is to explain the system direction, not to overstate product completion.

---

## Core Thesis

Most agents try to be:

```text
better at answering
better at calling tools
better at autonomous execution
```

agent+++ tries to be:

```text
better at moving projects forward
better at organizing multi-threaded AI labor
better at preserving evidence
better at stage convergence
better at long-term memory
```

If a normal agent is an execution assistant, agent+++ aims to become:

> an AI project operating system that turns fragmented ideas into long-running project assets.
