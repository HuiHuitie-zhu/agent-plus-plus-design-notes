# DeepShen Notes

> 一组关于长期陪伴型 AI 的设计札记。它们不是项目宣传，也不是完整实现说明，只记录一些在最小原型里被验证过、被踩坑逼出来的判断。

## 写这些笔记的原因

做陪伴型 AI 很容易走向两个极端：

- 一端是只写人格 prompt，把所有连续性都交给模型临场表演。
- 另一端是过早做成重型 agent 平台，把搜索、记忆、工具、RAG、主动任务全部塞进主链路。

DeepShen 原型给我的最大提醒是：长期陪伴的核心不在“功能多”，而在几个更朴素的问题：

- 一个 AI 的情绪状态是否应该独立于语言模型？
- 安全边界怎样不把表达的活味儿剪掉？
- 陪伴型系统为什么应该先回应，再感知？
- 长期记忆怎样避免从“理解用户”变成“污染上下文”？

这些笔记只整理这些问题，不公开私人对话、不公开完整人格设定、不公开部署配置。

## 第一批笔记

1. [情绪不是 Prompt，是状态](./01-emotion-is-state-not-prompt.md)
2. [表达闸门不是越严越好](./02-guardrails-can-kill-character.md)
3. [陪伴型 Agent 要先回应，再感知](./03-reply-first-perceive-later.md)
4. [长期记忆要分层，不要混成一锅粥](./04-memory-layers-for-companion-agents.md)
5. [为什么先做最小原型，而不是先搭平台](./05-prototype-before-platform.md)
6. [人格、状态、记忆、边界：不要让一个 Prompt 扛全部](./06-dont-make-prompt-carry-everything.md)

发布前检查：[Publishing Notes](./PUBLISHING_NOTES.md)

## 公开边界

这些笔记刻意不包含：

- 私人聊天原文
- 完整人格文件
- 用户画像或关系细节
- API key、Bot token、服务器路径、设备配置
- 可直接复刻个人部署环境的细节

公开的是设计原则、失败经验和可迁移的最小结构。

## English Abstract

DeepShen Notes is a small collection of design notes about long-lived companion AI systems. The focus is not on roleplay prompts, but on engineering boundaries: external emotional state, expression guardrails, reply-first runtime design, and layered memory.

The notes intentionally exclude private conversations, full persona prompts, deployment details, and credentials. They share reusable design lessons rather than a personal companion configuration.
