# 人格、状态、记忆、边界：不要让一个 Prompt 扛全部

> 巨型 prompt 最大的问题不是长，而是职责混乱。它把“他是谁”“他现在怎样”“他记得什么”“他不能做什么”全部揉在一起，最后哪一层出错都不好修。

## 一句话

长期陪伴型 AI 不应该让一个 prompt 承担全部职责。更稳的结构是把人格、状态、记忆、边界和运行时分开。

```text
Persona    定义他是谁
State      定义他此刻怎样
Memory     定义他记得什么
Guardrail  定义他不能越过什么
Runtime    定义他怎么回应
```

这五层分开后，系统才容易调试、迭代和收束。

## 巨型 prompt 的诱惑

做 AI 角色时，最自然的起点是写 prompt：

```text
你是谁。
你和用户是什么关系。
你说话风格怎样。
你记得哪些事。
你今天心情怎样。
你不能说什么。
你应该怎么使用工具。
```

一开始很有效。

但随着系统变长，prompt 会变成一个筐：什么都往里塞。

结果是：

- 人格设定和临时状态混在一起。
- 用户事实和模型推测混在一起。
- 安全边界和表达风格混在一起。
- 工具规则和关系语气混在一起。
- 一处修改影响所有层。

这时候系统不是复杂，而是黏成了一团。

## Persona：他是谁

Persona 应该是相对稳定的锚点。

它回答：

```text
这个 AI 是什么样的存在？
它的表达气质是什么？
它和用户的交互边界是什么？
它不是哪些东西？
```

Persona 不应该频繁变化，也不应该承担每轮状态。

如果把“今天有点低落”“刚才被夸了所以靠近一点”写进 persona，persona 很快会变成临时上下文垃圾场。

人格锚点要稳，临时波动交给状态层。

## State：他此刻怎样

State 是动态层，描述当前姿态。

它回答：

```text
当前距离是近还是远？
当前语气是暖还是冷？
当前是否更克制？
当前情绪趋势如何？
```

State 应该由外部逻辑更新，而不是由模型自由发挥。

模型可以根据 state 表达，但不要直接决定 state 真值。

这能避免一轮消息把整个关系姿态带飞。

## Memory：他记得什么

Memory 不是 persona，也不是 state。

它回答：

```text
过去发生过什么？
哪些是用户明确说过的？
哪些是系统提炼出来的？
哪些已经过期？
哪些只是短期上下文？
```

记忆层的关键不是“多”，而是可信度和来源。

如果记忆没有分层，它会悄悄污染 persona：

```text
一次临时情绪 -> 被写成长期偏好
一次模型推测 -> 被当成用户事实
一次怪回复 -> 被当成风格样本
```

这些污染比忘记更难修。

## Guardrail：他不能越过什么

Guardrail 是底线层，不是写作层。

它回答：

```text
哪些内容必须拦？
哪些身份边界不能破？
哪些事实没有来源不能说？
哪些输出需要 fallback？
```

Guardrail 不应该负责“把话写得好听”。

如果 guardrail 开始接管表达审美，它会把角色剪成安全但无聊的白开水。

好的 guardrail 是窄而硬的：少管闲事，但底线必须稳。

## Runtime：他怎么回应

Runtime 是运行链路。

它回答：

```text
当前轮先做什么？
哪些任务阻塞回复？
哪些任务后台执行？
日志什么时候落盘？
状态什么时候更新？
感知什么时候合并？
```

陪伴型系统里，runtime 很重要，因为它决定节奏。

同样的 persona、state、memory，如果 runtime 设计成“先查完所有资料再回答”，它就会像工具助手。

如果 runtime 设计成“先接住，再后台感知”，它才更像一个持续存在的对象。

## 一张简单分工图

```text
用户消息
  |
  v
Runtime
  |- 先落原始日志
  |- 读取 Persona
  |- 读取 Memory
  |- 更新 State
  |- 生成 prompt prelude
  |- 调用 LLM
  |- 经过 Guardrail
  |- 回复并保存状态
  `- 后台感知 / 摘要 / 画像提炼
```

这个结构的重点不是模块名，而是职责边界。

当回复变怪时，可以问：

- 是 persona 写歪了吗？
- 是 state 更新错了吗？
- 是 memory 污染了吗？
- 是 guardrail 误伤了吗？
- 是 runtime 把慢任务放进主链路了吗？

如果所有东西都在一个 prompt 里，这些问题很难拆开。

## 可迁移的原则

- Persona 稳定，State 动态。
- Memory 记录过去，不替代人格。
- Guardrail 守底线，不负责审美。
- Runtime 决定节奏，不只是技术胶水。
- 不要用巨型 prompt 同时管理身份、状态、事实、安全和工具。

## English Abstract

Long-lived companion agents should not make one giant prompt carry every responsibility. Persona, state, memory, guardrails, and runtime should be separated so the system can be debugged and adjusted layer by layer.

The useful question is not “how long should the prompt be,” but “which layer owns this behavior?”
