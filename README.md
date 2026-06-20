# agent+++ Public Design Notes / 公开设计笔记

agent+++ is a local-first project-oriented agent operating system. It uses a deep Thinker for project-level decisions, Flash execution agents for parallel work, Observation Contract for faithful feedback, and global/project memory for long-running continuity.

agent+++ 是一个本地优先的项目型 Agent 操作系统：用深度 Thinker 做项目决策，用 Flash 执行层并行推进，用 Observation Contract 保证反馈不失真，用全局沉淀层和项目记忆支持长期项目继续。

This repository contains public design notes only. It does not include private source code, credentials, leaked prompts, or production-ready sandbox code.

本仓库只包含公开设计笔记，不包含私有源码、密钥、泄漏提示词或生产级沙箱代码。

---

## What to Read First / 先看什么

### Core Architecture / 核心架构

1. Overview: [中文](agent-plus-plus-value-points.md) / [English](agent-plus-plus-value-points.en.md)
2. Implementation Status & Roadmap: [中文](implementation-status-and-roadmap.md) / [English](implementation-status-and-roadmap.en.md)
3. Observation Contract: [中文](observation-contract.md) / [English](observation-contract.en.md)
4. Global Precipitation Memory: [中文](global-precipitation-memory.md) / [English](global-precipitation-memory.en.md)
5. Thinker Architecture: [中文](thinker-architecture.md) / [English](thinker-architecture.en.md)
6. Why DeepSeek: [中文](why-deepseek.md) / [English](why-deepseek.en.md)

English versions are concise companions to the Chinese originals, not always line-by-line translations.

英文版是中文原文的精简 companion，不保证逐段等长翻译。

### Practical Experience 实战篇

7. Eight Lessons: [中文](eight-lessons.md) — hard-earned lessons from building agent+++ for a month
8. Safety Control Planes: [中文](safety-control-planes.md) — around 255 lines of deterministic safety for agent bash execution
9. Eval Practice: [中文](eval-practice.md) — Signal-based scoring for personal-project evaluation
10. Project Retrospective: [中文](project-retrospective.md) — Why I built and stopped agent+++ in 30 days
11. Final Fantasy: [中文](final-fantasy.md) — Distributed multi-node Thinker collaboration network

---

## Architecture Map / 架构地图

The core idea is not a single agent loop, but a separation of cognitive roles:

agent+++ 的核心不是单个 Agent loop，而是把认知角色拆开：

```text
Thinker -> project-level decisions and decomposition
Reflex Layer -> bounded execution and tool use
Observation Contract -> faithful tool-output reconstruction
Global / Project Memory -> long-running continuity and project assets
```

```text
Thinker 负责项目级判断和拆解
Reflex Layer 负责受控执行和工具调用
Observation Contract 负责把工具输出重建为可靠观察
Global / Project Memory 负责长期连续性和项目资产化
```

---

## Model Strategy / 模型策略

> DeepSeek is the default model base: Flash handles high-frequency execution and feedback processing, while Pro handles deep project decisions, convergence checkpoints, and generational handoff. The architecture is designed for high token utilization: raw logs do not go directly into Thinker, child threads inherit decision skeletons instead of full context, and long projects continue through staged handoff rather than endless context compression.

DeepSeek 是默认模型底座：Flash 用于高频执行和反馈处理，Pro 用于深度项目决策、阶段收束和代际切换。agent+++ 的 token 利用策略不是简单压缩，而是让原始日志不直接进入 Thinker，让子线程继承决策骨架而不是完整上下文，让长期项目通过阶段交接继续推进。

---

## Status / 状态

These are public design notes, not a claim that every feature is fully productized.

这些是公开设计笔记，不代表所有功能都已经完整产品化。

## License / 许可

Documentation is released under [CC BY 4.0](LICENSE).

文档按 [CC BY 4.0](LICENSE) 许可发布。
