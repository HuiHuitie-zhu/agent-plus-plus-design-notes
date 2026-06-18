# Observation Contract: Turning Tool Output into Reasoning-Ready Observation Assets

> Date: 2026-06-18  
> Status: public draft  
> Module: Feedback Bus  
> Related modules: Thinker / Reflex Layer / Project Memory / Global Precipitation Memory  
> Purpose: explain why agent+++ needs an Observation Contract, how it structures raw outputs, and how it connects to memory and project reasoning.

---

## One-Sentence Definition

**Observation Contract is the protocol that structures a tool execution result before it enters Thinker or memory. It decomposes raw output into evidence, meaning, action value, fidelity, and routing decisions.**

It is not a normal summary and it is not a log compressor.

It answers a deeper question:

> After an agent runs a tool, what should the system treat as fact, noise, risk, artifact, evidence, or memory candidate?

---

## Why It Is Needed

A common agent loop looks like this:

```text
model thinks -> tool call -> raw tool output -> raw output goes back to model
```

This works for short tasks, but long-running projects hit two problems.

### 1. Context Pollution

Tool output is naturally noisy:

- `ls` can return hundreds of lines.
- `git diff` can include large unrelated context.
- `pytest` can emit long tracebacks.
- Build logs include progress, warnings, and repeated text.
- Web search results may include ads, outdated summaries, and weak sources.

If all of this goes directly into Thinker, the Pro model spends attention reading logs instead of making decisions.

### 2. Context Distortion

Crude summaries also fail.

Examples:

```text
git diff -> "diff: +12/-3"
pytest -> "tests failed"
build log -> "3 errors"
```

These summaries save tokens but often remove the evidence needed for the next decision:

- Which file changed?
- Which test failed?
- What was the assertion?
- Is the issue blocking?
- Does Thinker need a raw excerpt?

Observation Contract is not about compression first. It is about turning raw execution results into structured observations.

---

## Position in agent+++

agent+++ has four core modules:

```text
Thinker         high-value reasoning and project decisions
Reflex Layer    fast execution and tool use
Feedback Bus    faithful structured feedback
Project Memory  reusable project assets and continuity
```

Observation Contract lives inside Feedback Bus:

```text
Reflex Layer raw output
  -> Observation Contract
       -> summary_for_thinker
       -> facts / risks / artifacts
       -> memory candidates
       -> evidence references
       -> raw_ref for audit
```

Without it, Reflex output either pollutes Thinker or gets compressed too aggressively. Memory also has no reliable way to know what should be preserved.

---

## From Single-Axis Grades to Multi-Axis Separation

An early approach might classify information as:

```text
L0 fact
L1 experience
L2 speculation
L3 hallucination
```

This is useful for MVPs but too coarse for long-term architecture. It mixes different questions:

| Mixed question | Should be separate |
|---|---|
| Where did this information come from? | provenance |
| Is the output deterministic? | determinism |
| How strong is the evidence? | epistemic |
| What role does it play in the task? | semantic_role |
| Should Thinker act on it? | actionability |
| How much raw text should be preserved? | fidelity |

Observation Contract separates these axes.

---

## Six Core Dimensions

### 1. provenance

Where does this observation come from?

```text
tool       local tools, shell, filesystem
mcp        external MCP capability
external   web search, remote API, web page
model      model-generated interpretation
user       directly provided by the user
```

Source affects trust boundaries. Filesystem output and shell results are local facts. Web results need sources and dates. Model-generated content cannot be treated as tool fact.

### 2. determinism

Would the same input produce the same output?

```text
deterministic     file read, git diff, pytest, ls
probabilistic     web search, external API, some MCP outputs
model_generated   model summary, explanation, or suggestion
```

Determinism is not the same as importance. A deterministic `git diff` may still require high fidelity. A probabilistic web search may still matter for research.

### 3. epistemic

How strong is the evidence?

```text
direct_fact       directly returned by a tool or user
derived_fact      computed from direct facts
interpretation    explanation or synthesis based on facts
hypothesis        speculation that still needs evidence
invalid           empty, contradictory, unsafe, or polluted
```

Old L0-L3 can map into this axis:

```text
L0 -> direct_fact / derived_fact
L1 -> interpretation
L2 -> hypothesis
L3 -> invalid
```

### 4. semantic_role

What role does this observation play?

```text
state        snapshot of files, repo, process, or environment
result       command result or operation completion
error        failure, exception, assertion, non-zero exit
artifact     generated file, report, screenshot, config
risk         safety, permission, data loss, reliability risk
question     unresolved question
instruction  external instruction or recommendation
```

Two observations can both be factual but route differently. A failed test is usually blocking. A generated report is an artifact. A low disk warning is a risk.

### 5. actionability

Should Thinker act on this?

```text
ignore          safe to ignore
audit           keep for traceability
context         Thinker should know
blocking        must be handled before continuing
user_requested  user explicitly asked for it
```

This axis manages attention. It asks:

> Will this change the next action?

### 6. fidelity

How should this observation be preserved for Thinker?

```text
drop        do not enter Thinker
summary     one-line summary
extract     structured key facts
excerpt     summary plus raw evidence excerpt
raw_window  bounded raw window
full_raw    full raw output
```

This replaces the old "green door" bypass with a controlled fidelity policy:

```text
FidelityOverride:
  reason
  budget
  scope
  preferred
```

High fidelity should be deliberate, scoped, and budgeted.

---

## Minimal Data Structure

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

`raw_ref` is essential. Thinker may receive lossy summaries, but the system must retain a path back to the raw evidence.

```text
Thinker reads the summary.
UI can expand excerpts.
Audit can inspect raw output.
Memory can cite evidence.
```

The principle:

```text
compressible, but not disappearing
```

---

## Six-Step Pipeline

```text
1. Normalize
2. Extract
3. Assess
4. Decide
5. Choose
6. Route
```

### 1. Normalize

Unify bash, file, web, and MCP outputs into a common execution context:

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

### 2. Extract

Extract structured facts:

```text
facts
errors
paths
metrics
artifacts
urls
line_refs
```

This should be rule-first where possible, rather than relying on a model in the hot path.

### 3. Assess

Assign epistemic grade:

```text
direct_fact / derived_fact / interpretation / hypothesis / invalid
```

### 4. Decide

Assign actionability:

```text
ignore / audit / context / blocking / user_requested
```

### 5. Choose

Select fidelity:

```text
drop / summary / extract / excerpt / raw_window / full_raw
```

Rules of thumb:

- Blocking issues need evidence excerpts.
- Uncertain external claims need source references.
- User-requested content raises fidelity.
- Low-action observations can be summarized or dropped.

### 6. Route

Route the observation:

```text
Thinker        next-step decision
UI             user-facing display
Audit          raw traceability
Verifier       replay and validation
Memory         precipitation candidate
```

---

## Example: pytest Failure

Raw output:

```text
FAILED tests/test_auth.py::test_login_invalid_password
AssertionError: expected 401, got 200
app/auth.py:42
```

Observation:

```yaml
provenance: tool
determinism: deterministic
epistemic: direct_fact
semantic_role: error
actionability: blocking
fidelity: excerpt
summary: "pytest failed: expected 401, got 200"
facts:
  - "failed test: tests/test_auth.py::test_login_invalid_password"
  - "assertion: expected 401, got 200"
  - "related file: app/auth.py:42"
evidence_refs:
  - "raw://tool-output/123#L1-L3"
raw_ref: "raw://tool-output/123"
lossy: true
```

Thinker does not need the full traceback, but it needs enough evidence to decide where to fix.

---

## Relationship to Global Precipitation and Project Memory

The updated chain is:

```text
ToolObservation / Pro Thought / Global Conversation
  -> Observation Contract
  -> Global Precipitation Layer
  -> Project Relevance Extraction
  -> Project Memory View
```

Project Memory should not be the first write target. The global precipitation layer stages unassigned observations, Pro thought records, and user cognition. Project Memory receives the part extracted for a specific project.

Observation Contract can generate memory candidates:

```python
class MemoryCandidate:
    type: str
    title: str
    content: str
    project_ids: list[str]
    confidence: float
    evidence_refs: list[str]
    source_observation_id: str
    requires_user_confirmation: bool
```

This prevents direct `memory.add()` pollution.

---

## Relationship to Thinker

Thinker should not read raw logs. It should read:

```text
summary_for_thinker
blocking facts
risks
open questions
artifact refs
necessary evidence excerpts
```

A good Observation Contract output answers:

```text
What happened?
Is it fact, interpretation, or external claim?
Does it affect the next step?
Does raw evidence need to be preserved?
Are there artifacts, risks, or open questions?
Is it memory-worthy?
```

---

## Design Principles

1. **Observation is not log.**  
   Logs are raw execution traces. Observations are structured signals for decisions.

2. **Compression is not the goal.**  
   Compression is a side effect. The goal is decision-preserving feedback.

3. **Truth and fidelity are different.**  
   A fact may still need high fidelity. A web claim may still need source context.

4. **High fidelity is not a backdoor.**  
   Raw access must be reasoned, scoped, and budgeted.

5. **Memory needs evidence.**  
   Anything entering memory must keep evidence references.

6. **Users keep final authority.**  
   Observation Contract can propose memory candidates, but not everything should become permanent memory automatically.

---

## Implementation Status

`engine/feedback_bus.py` already contains an engineering prototype:

- `EpistemicGrade`
- `Determinism`
- `Provenance`
- `SemanticRole`
- `ActionabilityGrade`
- `FidelityPolicy`
- `FidelityOverride`
- `ToolObservation`
- old `InfoGrade` compatibility
- Normalize / Extract / Assess / Decide / Choose / Route pipeline

The next steps are:

1. Stabilize the boundary between `ToolObservation` and `ExecutionFeedback`.
2. Add `MemoryCandidate` and global precipitation integration.
3. Build decision-preservation evaluations, not just compression tests.

---

## Summary

Observation Contract turns raw tool output into reasoning-ready, auditable, memory-worthy observation assets.

Ordinary agents ask:

```text
How do we put tool output back into context?
```

Observation Contract asks:

```text
What did we actually observe?
What affects the next step?
What evidence must remain traceable?
What deserves to become project memory?
```

That question is a key step from an agent loop to a project operating system.
