# 手搓 Agent 一个月，我踩了这 8 个坑

> 一个月前我觉得三天搓个 Agent 工作台挺牛的。
> 一个月后我觉得当时想得实在太简单了。
> 但这一个月踩的坑，比之前一年读论文都值。

---

## 写在前面

我做了个项目叫 agent+++，定位是"本地优先的项目教练型 Agent 工作台"。从搭骨架到勉强能跑，花了 3 天。然后花了剩下的 27 天修它在真实使用中暴露出来的问题。

这 8 个坑，每一个都是代码写进去、bug 跑出来、盯着屏幕想了半天才想通的。没有一个是读论文能读出来的。

---

## 坑 1：os.chdir() 是并行执行的隐形毒药

**场景：**
Eval 评测框架搭好了，并行跑 4 个测试用例。结果出来——3 个通过，1 个诡异失败。失败的那个在 `/tmp/eval_case_B/` 目录下跑，但它写出来的文件跑到了 `/tmp/eval_case_A/` 里去。

**根因：**
追了半天发现是 `os.chdir()`。

```python
# 某段代码里：
os.chdir(case_dir)
# 执行任务...
os.chdir(original_cwd)
```

`os.chdir()` 修改的是**进程全局 CWD**（Current Working Directory）。不是线程级、不是协程级——是进程级。

四个协程并发跑，A 切到 A 目录 → B 切到 B 目录 → A 写文件写到 B 的目录里——因为 CWD 被 B 改了。

**修复：**

```python
# 用 subprocess 的 cwd 参数
subprocess.run(cmd, cwd=case_dir)

# 或者用绝对路径拼接
file_path = os.path.join(case_dir, filename)

# 实在避不开 chdir 的，上锁
async with chdir_lock:
    os.chdir(case_dir)
    # 执行
```

修复完了加了一条项目纪律：**任何新增并发代码，先 grep 全项目确认没有裸 `os.chdir()` 调用。**

> **一句话教训：** `os.chdir()` 是进程级毒药，不是"临时切一下目录"的方便工具。

---

## 坑 2：WebSocket 自己等自己开门

**场景：**
Agent 要写一个文件，前端弹出授权弹窗，用户点了"允许"——然后凉了。没有任何响应。用户又点了一次，还是没反应。

**根因：**
这是经典到可以写进教材的异步死锁。

```python
# 伪代码——问题版本
async def websocket_loop(ws):
    async for msg in ws:
        if msg.type == "chat.send":
            # await 会阻塞整个接收循环！
            await handle_chat(msg.payload)
        elif msg.type == "confirm.response":
            # 永远收不到这条消息
            handle_confirm(msg.payload)
```

WebSocket 主循环收到 `chat.send` 后 `await handle_chat(...)`——而 `handle_chat` 内部的 Agent 又在等 `confirm_events[tool_call_id]` 这个事件被设置。但谁设置它？需要前端发 `confirm.response`。谁处理 `confirm.response`？**同一个 WebSocket 接收循环。** 被阻塞了。

自己在等自己开门。

**修复：**

```python
# 伪代码——修复版
async def websocket_loop(ws):
    async for msg in ws:
        if msg.type == "chat.send":
            # 后台任务执行，主循环继续收消息
            asyncio.create_task(handle_chat(msg.payload))
        elif msg.type == "confirm.response":
            set_confirm_event(msg.payload)
```

`chat.send` 启动后台 task 执行，主循环继续活着处理 `confirm.response`、`interrupt` 等控制消息。

**附加发现：** 用户聊天里回复"确认"也不会触发授权通过——只有点 UI 弹窗按钮才算。这个我们一开始也没意识到，测试了才发现"确认"这个词在聊天里只是普通消息，不经过 `confirm.response` 通道。

> **一句话教训：** WebSocket 接收循环里的 await，等于自己把自己锁门外。

---

## 坑 3：file_patch 遇到文件末尾没换行符就炸

**场景：**
Agent 要对一个配置文件做 patch 修改——简单的两行替换。生成的 unified diff 看起来完全正确。但 `file_patch` 工具返回"上下文不匹配"错误。

检查了好几遍：文本没问题，行号没问题，上下文也能对上。

**根因：**
文件最后一行没有换行符。

`splitlines(keepends=True)` 后，最后一行末尾没有 `\n`——但 patch 里的最后一行默认带着 `\n`。肉眼看起来一模一样，但字节对不上。

```python
# 文件内容（没有最后换行）：
"line1\nline2\nline3"
# splitlines(keepends=True) 后：
["line1\n", "line2\n", "line3"]
# 但 patch 期望的是：
["line1\n", "line2\n", "line3\n"]  # 末尾多了个换行
```

**修复：**
打 patch 前先用 `cat -e` 或 `xxd` 确认文件末尾字节。简单替换走 `file_edit` 而非 `file_patch`——edit 按行号替换，不依赖上下文匹配。

给 `file_patch` 加了个 helper：遇到文件末尾无换行符时自动补一个空行再做 patch，匹配成功后再把多余的空行删掉。

> **一句话教训：** 文本可视化没问题的东西，字节可能对不上。`cat -e` 是你的朋友。

---

## 坑 4：纯计数文本被当成事实存进记忆

**场景：**
Agent 执行了一段代码修改，沉淀管线自动提取"事实"。然后我发现记忆库里有这么一条：

> "共 1 行输出"

**这是一条"事实"。** 存在项目记忆里。下次 Agent 回忆相关项目经验的时候，会读到"共 1 行输出"——完全没有信息价值，但确实被当作事实存了。

**根因：**
代码里有个 `_summarize_file_ops` 函数，负责总结文件操作结果。它输出类似"共修改 N 个文件，新增 N 行"的统计文本。这些文本被直接投喂到事实提取管线——而管线没有过滤"纯统计文本"的规则。

统计文本没有动词核心，不是事实性断言。"单行输出 N 项" = 统计，不是事实。

**修复：**
加了双层过滤：

1. **L0：正则白名单**——已知的统计模式直接过滤
2. **L0.5：无动词检测**——候选事实的标签集如果没有任何动词核心，直接降级丢弃

```python
# 核心逻辑
if not has_verb_core(candidate_fact):
    drop_to_F0(candidate_fact)  # F0 = DROP
```

> **一句话教训：** 不是所有输出都值得沉淀。计数不是事实，统计不是知识。

---

## 坑 5：去重 key 粒度不够，同一事实存了 8 遍

**场景：**
同一段会话里，Agent 连续调了 3 次相同的工具查同一个文件，返回了相同的结果。沉淀管线提取出 3 条一模一样的"事实"。

**根因：**
去重 key 用的是 `(tool_call_id, fact_text)`——看起来挺合理。但问题在于：**同一工具的不同次调用 `tool_call_id` 不同**——所以一模一样的事实，因为 `tool_call_id` 变了就轻松过检。

核心缺失是**时间窗口维度**。

**修复：**

```python
# 去重要素增加到三个：
# session_id + fact_text + tool_name
# 再加时间窗口：同一 session 内、同一 tool_name、同一 fact_text
#   在 30 秒内只保留一次
def dedup_key(fact, session_id):
    return (session_id, fact.tool_name, fact.text)

def temporal_coalescer(facts, window_seconds=30):
    # 同 key 且在时间窗口内 → 合并
    ...
```

去重的检查点也要前置——不是在"入库时"，而是在"merge_candidates 时"。越早去重，下游所有逻辑都不出错。

> **一句话教训：** 去重不加上时间窗口，就等于没去重。

---

## 坑 6：自然语言提取用纯正则 = 大海捞针

**场景：**
工具返回了 800 字的长文本——设计决策、调研结论、用户表达的想法。事实提取管线跑完，只抓到了 3 条：

- "文件大小：1024KB"
- "修改文件：3 个"
- "耗时：2.3s"

而真正有价值的信息——"用户决定用 PostgreSQL 而非 MongoDB"、"性能瓶颈在 I/O 不在 CPU"——一条都没抓住。

**根因：**
事实提取管线当时只用了正则规则——`\d+` 抓数字、`git diff` 解析抓行号。对结构化输出（文件信息、统计数据）表现不错，但对**自然语言描述的工具输出**（设计决策、调研结论、用户意图）零提取能力。

引入 LLM 辅助提取的想法提出来了——但全线调 LLM 的成本不可接受。

**修复：**
设计了**三级提取架构**，核心约束是：**沉淀管线总额外成本 ≤ 执行管线成本的 5%。**

```
T1: 纯规则（0 成本）
    → 正则抓结构化数据
    → 覆盖 60% 场景

T2: 本地小模型分类器（~0.01 秒/次）
    → Qwen2.5-1.5B via Ollama
    → 只干一件事：判断"这段输出有值得沉淀的信息吗？"
    → 二元分类 + 置信度
    → 置信度低于阈值的不进下一级

T3: 复用 Pro Thought（0 额外成本）
    → Thinker 每轮的 Pro Thought 本就是沉淀提案的天然载体
    → 把 T2 分类为 positive 的工具输出注入 Pro Thought 的判断逻辑
    → 不新增额外 LLM 调用
```

三级递进：绝大多数文本在 T1 就处理完了，少量需要 T2 筛选，极少数需要 T3 深度提取。

> **一句话教训：** 正则不能提取自然语言。但不是什么都值得调 LLM。三级成本分级才是正解。

---

## 坑 7：DeepSeek 特化不是"能调用 V4"，是每轮都知道该不该 thinking

**场景：**
项目初期对接 DeepSeek V4 的时候，我们用 OpenAI-compatible client 写了套通用封装——改个 `base_url`、改个 `model_name`，以为"接上了"。

跑起来确实能出结果。但跟同配置的 Claude 对比，效果差一截。

**根因：**
OpenAI-compatible 封装是起点，不是终点。DeepSeek V4 有很多特性没法通过通用封装发挥：

- **thinking mode**：不是所有轮次都需要深度思考——Pro 模型的 thinking token 很贵。需要在调度层判断"这轮该不该 thinking"。
- **strict schema**：DeepSeek 的 strict 模式对工具调用稳定性提升明显，但需要所有 object 字段 `required` + `additionalProperties:false`——通用封装不会帮你做这个转换。
- **context cache**：DeepSeek 的缓存策略跟 OpenAI 不同——`prefix` 结构的缓存命中率直接影响成本和速度。
- **模型路由**：什么时候用 Pro（深度决策）、什么时候用 Flash（高频执行）——不是固定配置，而是运行时动态判断。

**修复：**
做了一层 `ModelRouter`，负责运行时判断：

```python
def suggest_route(intent, context_size, tool_complexity):
    if intent in ("goal_decomposition", "convergence", "architectural"):
        return "deepseek-v4-pro", {"thinking": "on"}
    elif context_size > 80000:
        return "deepseek-v4-flash", {"thinking": "off"}
    elif tool_complexity > 5:
        return "deepseek-v4-pro", {"thinking": "on"}
    else:
        return "deepseek-v4-flash", {"thinking": "off"}
```

工具 schema 全部按 strict 模式重写——每个 object 字段 `required` + `additionalProperties:false`。肉眼看着啰嗦，但模型调用稳定性确实上了一个台阶。

> **一句话教训：** 换模型不是改配置，是重新设计调度策略。OpenAI-compatible 的"兼容"只是协议层兼容——不影响你的代码跑不跑得起来，但影响跑得好不好。

---

## 坑 8：聪明单兵不等于作战系统

**场景：**
项目 MVP 跑通了——Agent 能调用工具、能安全执行 bash、能搜索、能读写文件。但总觉得跟 Claude Code 和 Codex 差距巨大——不是差一点，是差一个维度。

具体来说：
- 任务跑到一半断了，没有恢复机制
- 改了一大段代码结果不满意，回退不了
- 上下文塞到 50K token 就开始糊
- 有些测试环境下表现好，换个场景就拉胯，但不知道哪个场景拉胯

**根因分析：**
我们把一个 Agent 做得很聪明——但它没有配套的系统工程能力。差的东西列出来：

```
✅ 会调用工具
❌ 没有任务状态机 → 不可恢复
❌ 没有 checkpoint → 不敢改大代码
❌ 没有上下文预算 → 1M context 变垃圾堆
❌ 没有评测体系 → 只靠感觉
```

"会调用工具"让 Agent 能**做一些事**。"任务状态机 + checkpoint + 上下文预算 + 评测体系"让 Agent 能**连续做复杂的事**。

两者之间差了一个系统工程周期。

**修复不是一天的事，但我们定了一个立项前的问题清单：**

> 每个新能力立项前，问一遍：
> 1. 它是有状态的吗？
> 2. 它可回滚吗？
> 3. 它不占爆上下文吗？
> 4. 如何验证完成度？

如果你有一个答案是否定的，那就不是"后续优化项"，而是"当前阻塞项"。

> **一句话教训：** 一个聪明的 Agent 跟一个可靠的生产系统之间，差的不是智能，是工程。

---

## 最后

这 8 个坑回过头看，没有一个需要高深的技术来解决。每一个的解法都很朴素——有的只是加几行判断、换个工具函数、或者改个数据结构。

但难的是：**这些坑只有在真正做出来、用起来、跑崩了之后才能发现。**

读论文不会告诉你 `os.chdir()` 是并发的毒药。读文档不会告诉你 `splitlines(keepends=True)` 会在最后一行坑你。看别人的架构图不会告诉你 WebSocket 回环能自己等自己开门。

所以如果你也在自己搓 Agent——别怕踩坑，反正肯定要踩的。把这 8 个先记下来，能绕一个是一个。

---

写这篇的时候我在想：这些教训的价值，可能比项目本身还大。

项目代码放那可能没人看。但 `os.chdir()` 那件事，如果有人搜到了、绕过了，就值了。

*完整设计笔记（6 篇双语架构文档 + 安全系统 + 评测框架）：[agent-plus-plus-design-notes](https://github.com/HuiHuitie-zhu/agent-plus-plus-design-notes)*
