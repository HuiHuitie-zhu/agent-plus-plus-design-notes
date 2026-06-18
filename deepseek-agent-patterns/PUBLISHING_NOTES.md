# Publishing Notes

## Duplication Check

As of 2026-06-18, the public ecosystem mostly contains three adjacent categories:

1. Coding agents or IDE extensions that can connect to DeepSeek, such as Aider, Roo Code, and Continue.
2. General-purpose agent platforms such as OpenHands.
3. Research or experimental systems around context compression, prompt caching, memory folding, or sandbox checkpointing.

I did not find an obvious repository positioned specifically as a "DeepSeek-native agent engineering pattern library."

Recommended positioning:

> A practical pattern library for building DeepSeek-native agents.

Avoid positioning it as:

> Another desktop AI agent.

## Differentiation

- Not just "DeepSeek as a provider"; focus on DeepSeek API semantics.
- Explicit handling of `reasoning_content` in thinking + tool-call chains.
- Context-cache-aware prompt boundaries.
- Context budgets and compaction order for 1M-token agents.
- Tool safety attributes and file-edit trust layers.

## Risk Checklist

### High risk: non-public or leaked material

Do not publish:

- leaked source code
- full system prompts
- copied tool definitions
- definitive claims about third-party internal implementations

Safer wording:

- "Inspired by public agent engineering patterns"
- "Based on public documentation and implementation experience"
- "A practical pattern library"

### Medium risk: being mistaken for a production agent

State clearly near the top:

- documentation-first
- research prototype
- not a production sandbox
- no warranty

### High risk: privacy and credentials

Before publishing, remove or ignore:

```gitignore
.env
data/
*.log
__pycache__/
.pytest_cache/
frontend/node_modules/
frontend/dist/
```

### Medium risk: unsafe tool examples

If code examples are included, default to read-only behavior. File writes, bash execution, and desktop-control tools should be isolated examples with explicit warnings.

## Recommended Repository Description

English:

> Engineering patterns for DeepSeek-native agents: thinking, tool calls, context caching, context budgets, tool safety, and file-edit trust layers.

Chinese:

> DeepSeek-native Agent 工程模式库：thinking、tool calls、context cache、上下文预算、工具安全与文件编辑信任层。

## Recommended Topics

```text
deepseek
ai-agent
agent-patterns
llm-agents
tool-calling
context-caching
prompt-caching
coding-agent
agent-safety
```

## Suggested First Release

Start with README-only. Do not move the full app yet.

Then add:

1. minimal DeepSeek client
2. context budget module
3. tool safety schema
4. file checkpoint example
5. task lifecycle example
