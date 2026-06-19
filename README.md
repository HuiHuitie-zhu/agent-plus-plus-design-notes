# agent+++ Public Design Notes / 公开设计笔记

agent+++ is a local-first project-oriented agent operating system. It uses a deep Thinker for project-level decisions, Flash execution agents for parallel work, Observation Contract for faithful feedback, and global/project memory for long-running continuity.

agent+++ 是一个本地优先的项目型 Agent 操作系统：用深度 Thinker 做项目决策，用 Flash 执行层并行推进，用 Observation Contract 保证反馈不失真，用全局沉淀层和项目记忆支持长期项目继续。

---

## Reading Order / 阅读顺序

1. Overview: [中文](agent-plus-plus-value-points.md) / [English](agent-plus-plus-value-points.en.md)
2. Implementation Status & Roadmap: [中文](implementation-status-and-roadmap.md) / [English](implementation-status-and-roadmap.en.md)
3. Observation Contract: [中文](observation-contract.md) / [English](observation-contract.en.md)
4. Global Precipitation Memory: [中文](global-precipitation-memory.md) / [English](global-precipitation-memory.en.md)
5. Thinker Architecture: [中文](thinker-architecture.md) / [English](thinker-architecture.en.md)
6. Why DeepSeek: [中文](why-deepseek.md) / [English](why-deepseek.en.md)

---

## Model Strategy / 模型策略

> DeepSeek is the default model base: Flash handles high-frequency execution and feedback processing, while Pro handles deep project decisions, convergence checkpoints, and generational handoff. The architecture is designed for high token utilization: raw logs do not go directly into Thinker, child threads inherit decision skeletons instead of full context, and long projects continue through staged handoff rather than endless context compression.

DeepSeek 是默认模型底座：Flash 用于高频执行和反馈处理，Pro 用于深度项目决策、阶段收束和代际切换。agent+++ 的 token 利用策略不是简单压缩，而是让原始日志不直接进入 Thinker，让子线程继承决策骨架而不是完整上下文，让长期项目通过阶段交接继续推进。

---

## Status / 状态

These are public design notes, not a claim that every feature is fully productized.

这些是公开设计笔记，不代表所有功能都已经完整产品化。
