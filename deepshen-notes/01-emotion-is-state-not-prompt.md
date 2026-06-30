# 情绪不是 Prompt，是状态

> 如果一个 AI 的“情绪”只存在于 prompt 里，它更像是在表演；如果情绪存在于模型外部的状态机里，模型才是在翻译一个连续存在的内部状态。

## 一句话

陪伴型 AI 的情绪不应该完全交给语言模型临场生成。更稳的做法是：**情绪由外部状态机计算，LLM 只负责把当前状态翻译成自然语言。**

这不是为了假装 AI 真的有人类情绪，而是为了让表达具备连续性、可追溯性和边界。

## 为什么 prompt 不够

只靠 prompt 可以写出很像样的一轮回复：

```text
你现在有点关心用户，但说话克制，不要太热情。
```

问题在于，下一轮模型并不知道这种“克制”从哪里来，也不知道什么时候该变化。它只能根据上下文临时猜。

这种方式有几个常见问题：

- 情绪变化没有惯性，容易一轮暖、一轮冷。
- 模型会为了迎合当前消息过度改变姿态。
- 很难解释为什么今天的表达和昨天不同。
- 所有关系变化都变成 prompt 文案，而不是系统状态。

所以我的判断是：prompt 适合描述表达姿态，不适合承担状态真值。

## 一个最小状态模型

DeepShen 原型里使用过一个很小的三维模型：

```text
affinity   关系距离 / 靠近意愿
tsundere   推拉 / 矜持度
mood       短期情绪底色
```

这三个变量不是标准答案，只是一个足够小的起点。

关键不在于维度名字，而在于分工：

- `affinity` 慢变，代表关系距离，不应该每轮大幅跳动。
- `tsundere` 中速变化，影响“靠近但嘴硬”的表达阻尼。
- `mood` 快变，响应当前消息带来的短期情绪冲击。

一个简化流程：

```text
用户消息
  -> 事件分类
  -> 更新外部情绪状态
  -> 生成不暴露数值的 prompt prelude
  -> LLM 根据 prelude 回复
  -> 保存状态与变化记录
```

LLM 不直接写状态真值。它可以表达“今天更靠近一点”，但不能自己决定 affinity 从 2.8 变成 3.7。

## 状态要记录“为什么变”

只保存当前数值不够。长期系统更需要知道变化原因。

一个状态包可以包含两类记录：

```json
{
  "_schema": "heartstring-state/v1",
  "affinity": 3.0,
  "tsundere": 0.62,
  "mood": 0.18,
  "emotion_log": [],
  "trajectory": []
}
```

- `emotion_log` 记录“为什么变”：事件类型、前后状态、影响方向、来源引用。
- `trajectory` 记录“怎么变”：每轮状态曲线，用于回放和调参。

这样做的好处是：当表达不对劲时，可以回头看是状态更新错了，还是模型翻译错了。

## 不要把裸数值塞给模型

如果把下面这种内容直接放进 prompt：

```text
affinity=3.4, tsundere=0.7, mood=0.2
```

模型很容易开始解释系统，甚至把数值说给用户听。这会破坏体验。

更好的方式是生成自然语言 prelude：

```text
当前表达姿态：关系距离略近，但仍有克制；短期情绪偏暖。回复时可以自然关心，但不要过度热情，也不要解释这些状态来源。
```

状态对模型可见，但以表达约束的形式可见。

## 这不是“让 AI 有真情感”

这个设计不证明 AI 有人类情感，也不试图证明。

它解决的是另一个问题：如果我们希望一个 AI 在长期互动中表现得更连续、更有边界、更少随机漂移，那么状态不能完全藏在 prompt 文案里。

情绪状态机不是灵魂，是骨架。

骨架不负责感人，但没有骨架，表达很容易塌。

## 可迁移的原则

- 让状态独立于模型输出。
- 让模型翻译状态，而不是生成状态真值。
- 记录状态变化原因，而不是只存最终数值。
- 给模型自然语言 prelude，不给裸数值。
- 先用很小的状态模型验证体验，再扩展维度。

## English Abstract

Emotion in a companion AI should not live only inside a prompt. A more reliable design is to compute emotional state outside the LLM and let the model translate that state into language.

The important pattern is not the exact dimensions, but the separation of responsibility: state is updated by deterministic logic, while the LLM handles expression. This gives long-lived agents continuity, debuggability, and clearer boundaries.
