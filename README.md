# agent+++ Public Design Notes

This directory contains the public-facing design notes for agent+++.

- [中文版 README](README.zh-CN.md)
- [English README](README.en.md)

Recommended reading order:

1. [agent-plus-plus-value-points.md](agent-plus-plus-value-points.md)
2. [observation-contract.md](observation-contract.md)
3. [global-precipitation-memory.md](global-precipitation-memory.md)
4. [thinker-architecture.md](thinker-architecture.md)

Short positioning:

> agent+++ is a local-first project-oriented agent operating system. It uses a deep Thinker for project decisions, Flash execution agents for parallel work, Observation Contract for faithful feedback, and global/project memory for long-running continuity.

中文一句话：

> agent+++ 是一个本地优先的项目型 Agent 操作系统，用深度 Thinker 做项目决策，用 Flash 执行层并行推进，用 Observation Contract 保证反馈不失真，用全局沉淀层和项目记忆支持长期项目继续。

Model strategy:

> DeepSeek is the default model base: Flash is used for high-frequency execution and feedback handling, while Pro is used for deep project decisions, convergence checkpoints, and generational handoff. The architecture is designed for high token utilization: raw logs do not go directly into Thinker, child threads inherit decision skeletons instead of full context, and long projects continue through staged handoff rather than endless context compression.
