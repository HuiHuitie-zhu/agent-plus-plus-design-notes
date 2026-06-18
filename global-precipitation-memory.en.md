# Global Precipitation Memory: Global Staging and Project Memory Projection

> Date: 2026-06-18  
> Status: public draft  
> Module: Project Memory  
> Related modules: Observation Contract / Thinker / Feedback Bus  
> Purpose: explain agent+++'s memory architecture: global precipitation, project extraction, creative residue, and forgetting lifecycle.

---

## One-Sentence Definition

**Global Precipitation Memory is the global staging layer for agent+++: it is not project memory itself, but a lifecycle-managed pool for high-value observations, Pro thought records, user cognition, and unassigned cognitive residue.**

Project Memory is not the first storage layer.

More precisely:

> Project Memory is a project-scoped view projected from the global precipitation layer.

This shifts memory from "each project has its own isolated history" to "the system forms global cognition first, then projects the relevant part back into the current project."

---

## Value

This mechanism is valuable because it solves four problems that long-running agents inevitably face:

1. **Memory pollution**: model thoughts, tool logs, temporary guesses, and user preferences get mixed together.
2. **Project fragmentation**: cross-project lessons and user cognition are trapped inside individual projects.
3. **Unbounded accumulation**: global memory grows forever and becomes a context junkyard.
4. **Creative decay**: memory only serves factual reasoning, making the system increasingly conservative and dull.

The core shift:

```text
memory is not permanent storage;
memory is lifecycle-managed cognitive metabolism
```

Each memory has four possible destinies:

```text
claimed by a project
absorbed into user cognition
reused as creative residue
forgotten
```

---

## Architecture Position

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

Responsibilities:

- Observation Contract labels and classifies.
- Global Precipitation Layer stages unassigned cognition.
- Memory Projection Layer extracts relevant material for the current project and task.
- Project Memory View becomes the context Thinker can use.

---

## Why the First Layer Should Be Global

### 1. User Cognition Is Cross-Project

User preferences, decision style, communication habits, taste, and collaboration expectations do not belong to a single project.

Example:

```text
The user prefers direct automation chains such as shell / Python over over-abstracted workflows.
```

This should apply across projects.

### 2. Cross-Project Lessons Should Transfer

A lesson learned in one project may prevent failure in another.

Example:

```text
Hook registration changes may require a restart or /clear to take effect.
```

This is not only project-specific knowledge.

### 3. Project Memory Should Not Receive All Runtime Residue

If every observation is written directly into Project Memory, each project will accumulate:

- one-off errors
- temporary guesses
- obsolete designs
- transient file states
- unconfirmed model interpretations

Project Memory should hold project assets with clear ownership, not raw residue.

---

## Observation Contract as Memory Classifier

The global layer can contain many fragments. It needs classification.

Observation Contract provides that classification:

```text
provenance
determinism
epistemic
semantic_role
actionability
fidelity
```

These labels decide:

```text
Can this enter precipitation?
Should it be projected into a project?
Should it become user cognition?
Can it be used as factual context?
Can it become creative residue?
When should it be forgotten?
```

Observation Contract is therefore not only a Feedback Bus protocol. It is also a memory lifecycle classifier.

---

## Pro Thought Must Also Pass Observation Contract

Tool outputs need Observation Contract. Pro thought records need it too.

Reason:

> Pro reasoning can produce high-value judgment, but also explanations, assumptions, and unconfirmed guesses.

If Pro thought enters memory without labeling, the system may turn its own speculation into future "fact."

Default classification for Pro thought:

```yaml
source_type: pro_thought
provenance: model
determinism: model_generated
epistemic: interpretation or hypothesis
requires_confirmation: true
```

Only with tool evidence, user confirmation, or project documents should it be upgraded to:

```text
derived_fact
decision
lesson
project_fact
```

This prevents self-contamination:

```text
model guess -> memory -> future factual premise
```

---

## Main Chain: From Precipitation to Project Extraction

Before project-level Pro reasoning, the system extracts relevant material from the global layer:

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

This is not simple semantic retrieval. It is relevance extraction based on project, task, user intent, and reasoning phase.

Important rule:

```text
extracted material merges into Project Memory first,
then Project Memory enters Thinker context
```

Thinker receives one coherent project context, not two unrelated memory bags.

---

## No Backflow After Extraction

Once content is extracted from the global layer and merged into Project Memory, it should not return to the global layer as part of Project Memory.

Otherwise the system creates a memory echo chamber:

```text
global precipitation
  -> project memory
  -> Pro context
  -> Pro output
  -> precipitation again
  -> extracted again
```

Correct transition:

```text
active_precipitate
  -> extracted_to_project
  -> compact_tombstone
```

The full content is removed from the global layer. A tiny tombstone remains:

```yaml
memory_id: xxx
content_hash: xxx
extracted_to_project_id: xxx
extracted_at: 2026-06-18
```

The tombstone does not enter context or creative residue. It only prevents duplication and backflow.

---

## Global Conversation and User Cognition

After deep reasoning in a global conversation, the system should extract user cognition rather than project knowledge:

- long-term preferences
- communication style
- decision habits
- taste and aesthetic direction
- expectations for agent behavior
- boundaries and taboos
- repeated working patterns

Chain:

```text
Global Conversation
  -> Pro Deep Thinking
  -> Observation Contract
  -> User Cognition Candidate
  -> Global User Memory
```

Project Memory asks:

```text
How should this project continue?
```

Global User Memory asks:

```text
How does this user think, decide, and collaborate over time?
```

---

## Three-Stage Lifecycle

The global layer must not grow forever.

Unextracted memory moves through:

```text
Retention
  -> Hallucination
  -> Oblivion
```

### 1. Retention

New memory enters retention:

```yaml
lifecycle_stage: retention
strategy: preserve structure
purpose: wait for project extraction, user cognition, or cross-project reuse
```

Retention still carries factual responsibility and must follow Observation Contract evidence grades.

### 2. Hallucination

Long-unclaimed memory does not need to be deleted immediately. It can enter hallucination.

This stage is not for factual reasoning. It is for creative divergence.

```yaml
lifecycle_stage: hallucination
strategy: break apart, de-source, weakly recombine
purpose: brainstorming, naming, analogy, writing, style stimulation
```

Fragments can become:

- concept fragments
- metaphor fragments
- structural fragments
- style fragments
- question fragments
- unfinished ideas

Example:

```text
Original:
Observation Contract is a structured protocol before tool output enters Thinker and Memory.

Creative residue:
- observation is not log
- facts need passports
- customs before cognition
- memory crystallizes from residue
```

These fragments no longer carry factual responsibility. They are creative fuel only.

### 3. Oblivion

Hallucination memory must have a limit.

Low-value fragments eventually enter oblivion:

```yaml
lifecycle_stage: oblivion
strategy: delete low-reuse, low-novelty, repetitive fragments
purpose: control system entropy
```

Oblivion means permanent deletion.

---

## Safety Boundary for Hallucination

Hallucination is distinctive and useful, but it must be isolated from factual reasoning.

```text
code repair        forbidden
safety judgment    forbidden
architecture finalization default forbidden unless brainstorming
factual research   forbidden
creative ideation  allowed
naming/writing     allowed in small amounts
story framing      allowed
```

Hallucination is not permission to lie. It is a source of nonlinear creative material.

---

## Reference Data Structure

```python
class PrecipitatedMemory:
    id: str
    content: str
    source_type: str

    provenance: str
    determinism: str
    epistemic: str
    semantic_role: str
    actionability: str
    fidelity: str

    scope_hint: str
    project_affinity: dict[str, float]
    user_affinity: float
    confidence: float

    evidence_refs: list[str]
    source_observation_id: str | None
    source_thought_id: str | None

    lifecycle_stage: str
    created_at: str
    last_matched_at: str | None
    extracted_to_project_id: str | None
    tombstone_hash: str | None
```

The key is to separate:

```text
factual confidence
project relevance
lifecycle state
```

Traditional memory systems often collapse these into a single similarity score.

---

## Implementation Path

### Phase 1: Global Precipitation MVP

- Pro Thought and Tool Observation pass through Observation Contract.
- Generate `PrecipitatedMemory`.
- Default to retention.
- Preserve evidence references and source IDs.

### Phase 2: Project Extraction and Merge

- Extract relevant global memory before project Pro reasoning.
- Merge into Project Memory View.
- After extraction, remove full content and keep tombstone.

### Phase 3: Lifecycle Management

- Move long-unextracted retention memory into hallucination.
- Break hallucination memory into creative fragments.
- Move low-value fragments into oblivion.

### Phase 4: Creative Injection

- Inject hallucination fragments only in creative tasks.
- Disable for technical, factual, safety, and code-repair tasks.
- Track creative yield for future retention or deletion.

---

## Feasibility

Overall feasibility: **8/10**.

| Module | Feasibility | Main Difficulty |
|---|---:|---|
| Observation Contract for all inputs | 9/10 | Pro thought must not self-upgrade into fact |
| Global precipitation layer | 9/10 | Data model and storage are clear |
| Project relevance extraction | 7/10 | More than vector similarity |
| Project Memory merge | 8/10 | Deduplication, conflicts, overwrite rules |
| Tombstone after extraction | 8/10 | Prevent duplicate extraction and backflow |
| Retention / Hallucination / Oblivion | 7/10 | Lifecycle scoring needs tuning |
| Creative residue injection | 6.5/10 | Must be isolated from factual reasoning |
| Global user cognition | 8/10 | Requires confidence and confirmation policy |

---

## Summary

Global Precipitation Memory is agent+++'s cognitive metabolism system.

It lets Observation Contract govern not only tool feedback but long-term memory. It turns Project Memory from an isolated project notebook into a projected view from global precipitation. It lets unclaimed memory either become project knowledge, user cognition, creative residue, or forgotten.

Ordinary memory systems try to:

```text
remember more
```

Global Precipitation Memory tries to:

```text
know what should belong, what should ferment, and what should be forgotten
```
