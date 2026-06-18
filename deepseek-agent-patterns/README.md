# DeepSeek Agent Patterns

> Engineering patterns for DeepSeek-native agents: thinking, tool calls, context caching, context budgets, tool safety, and file-edit trust layers.

This is a documentation-first open-source package, not a full agent product. It collects engineering patterns that matter when building agents on DeepSeek V4, especially the parts that affect cost, stability, and tool-call correctness.

## Why This Exists

Many agent frameworks can connect to the DeepSeek API. Connection is not the same thing as good utilization.

DeepSeek V4 exposes several agent-relevant capabilities:

- Thinking mode is enabled by default and returns reasoning through `reasoning_content`.
- In thinking + tool-call flows, assistant messages that performed tool calls must preserve `reasoning_content` in later requests.
- Context caching is enabled by default and works best when stable prefixes are reused.
- API usage returns `prompt_cache_hit_tokens` and `prompt_cache_miss_tokens`, which can drive context orchestration.
- V4 Flash and V4 Pro are useful routing targets for different cost, latency, and reasoning-depth needs.

This project focuses on turning these API properties into agent architecture, instead of merely adding DeepSeek as another provider string.

## Who This Is For

- Developers building coding agents, desktop agents, or workflow agents with DeepSeek API.
- Teams with OpenAI-compatible agents who want better use of DeepSeek thinking, cache, and tool calls.
- Researchers and engineers interested in context management, tool safety, and file-edit rollback mechanisms.

## Core Patterns

### 1. DeepSeek-native LLM Client

Avoid treating DeepSeek as a thin OpenAI-compatible wrapper. A DeepSeek-native client should explicitly model:

- `thinking`: enabled / disabled
- `reasoning_effort`: high / max
- `reasoning_content`: streaming parse, persistence, and selective replay
- tool calls: streamed argument assembly and multi-turn tool-call chains
- cache metrics: hit / miss token extraction
- DeepSeek-specific errors: auth, balance, rate limits, context length, content filters, model availability, incompatible parameters

### 2. Reasoning Content Chain

In ordinary multi-turn chat, old `reasoning_content` does not need to be replayed. But if the assistant message triggered tool calls, its `reasoning_content` must be preserved along with the tool-call history.

Recommended rule:

```text
assistant without tool_calls:
  keep final content
  drop old reasoning_content during compaction

assistant with tool_calls:
  keep content
  keep reasoning_content
  keep tool_calls
  keep matching tool results
```

This is not merely a UI concern. It is protocol correctness.

### 3. Cache-friendly Prompt Boundary

DeepSeek context caching is sensitive to stable prefix reuse. Agent prompts should be split into:

```text
static prefix:
  identity
  safety rules
  behavior rules
  stable tool descriptions

dynamic suffix:
  retrieved memories
  current task state
  volatile tool results
  context warnings
```

Principle: keep stable content early and volatile content late. Do not insert per-turn state into the middle of the system prompt if you care about prefix caching.

### 4. Context Budget for 1M-token Agents

A 1M-token context window is not a trash bin. Large context windows make budgeting more important, not less.

A practical default budget:

| Segment | Ratio | Purpose |
|---|---:|---|
| system | 15% | identity, behavior, safety |
| tool definitions | 5% | stable tool schemas |
| history | 50% | conversation and tool results |
| current task | 10% | active request and working set |
| reserve | 20% | output and emergency headroom |

Recommended compaction order:

1. Drop old non-tool-call reasoning.
2. Truncate long tool outputs while preserving head and tail structure.
3. Fold old tool call / tool result pairs.
4. Summarize older conversation turns.
5. Drop raw text only as the last resort.

### 5. Tool Safety Attributes

Do not ask the LLM to decide whether a tool is safe at runtime. Tool safety should be part of the tool definition:

```python
ToolSafety(
    is_readonly=True,
    is_destructive=False,
    needs_confirmation=False,
    is_concurrency_safe=True,
    risk_level=1,
)
```

These attributes can drive:

- Plan mode by physically removing write tools.
- Parallel execution for read tools and serialized execution for write tools.
- Confirmation before high-risk operations.
- Safe filtering of tool lists and schemas.
- Audit logs and user-facing risk explanations.

The core idea: safety boundaries should be implemented by deterministic control planes wherever possible, not by hoping the model follows a prompt.

### 6. File Editing Trust Layer

Coding-agent file edits should not stop at "write succeeded." A minimal trust layer should include:

- pre-edit checkpoint
- diff preview
- post-confirmation hash check
- patch-first editing
- timeline
- rewind

Recommended flow:

```text
read current file
compute hash_before
generate diff preview
ask confirmation if needed
re-read file and verify hash_before
save checkpoint
apply edit or patch
record timeline event
return diff and checkpoint id
```

### 7. Model and Thinking Router

Model routing should be cheap and deterministic. Do not spend another LLM call deciding which LLM to call.

Example:

| Task | Model | Thinking |
|---|---|---|
| short chat / simple command | V4 Flash | off |
| normal agent task | V4 Flash | on |
| multi-tool / debugging / analysis | V4 Flash | deep / max |
| large repo analysis / high-stakes planning | V4 Pro | deep / max |

Routing inputs can include:

- request length and intent keywords
- estimated context tokens
- recent cache hit ratio
- tool-call round count
- retry/failure history

### 8. From Chat Loop to Recoverable Tasks

A reliable agent is more than a while-loop. Introduce a task lifecycle:

```text
task:
  pending -> running -> paused -> completed
                  |          |
                  v          v
                failed    resumed
```

Each task should preserve:

- user input
- current step
- model / thinking mode
- message snapshot
- last checkpoint id
- token and cost usage
- failure reason

This gives the agent interruption, recovery, long-task handling, and auditability.

## Suggested Repository Shape

```text
deepseek-agent-patterns/
  README.md
  README.zh-CN.md
  docs/
    01-deepseek-native-client.md
    02-context-budget.md
    03-tool-safety.md
    04-file-editing-trust-layer.md
    05-recoverable-agent-tasks.md
  examples/
    deepseek_client_minimal.py
    context_budget.py
    tool_safety.py
    file_checkpoint.py
```

## Non-goals

- This is not a complete desktop agent product.
- This repository does not include leaked source code, full prompts, or non-public material.
- This repository does not claim the internal implementation details of any third-party agent.
- This repository does not provide a production-grade sandbox.

## References

- DeepSeek Models and Pricing: https://api-docs.deepseek.com/quick_start/pricing
- DeepSeek Thinking Mode: https://api-docs.deepseek.com/guides/thinking_mode
- DeepSeek Context Caching: https://api-docs.deepseek.com/guides/kv_cache
- DeepSeek Function Calling: https://api-docs.deepseek.com/guides/function_calling
- Aider DeepSeek integration: https://aider.chat/docs/llms/deepseek.html
- Roo Code DeepSeek provider: https://docs.roocode.com/providers/deepseek
- Continue config reference: https://docs.continue.dev/reference
- OpenHands paper: https://arxiv.org/abs/2407.16741
- Prompt caching for long-horizon agentic tasks: https://arxiv.org/abs/2601.06007
- DeepAgent memory folding: https://arxiv.org/abs/2510.21618

## License

Recommended: MIT for code examples, CC BY 4.0 for documentation.
