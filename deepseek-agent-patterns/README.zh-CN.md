# DeepSeek Agent Patterns

> 面向 DeepSeek V4 的 Agent 工程模式：thinking、tool calls、context cache、上下文预算、工具安全与文件编辑信任层。

这是一个资料型开源包，不是完整 Agent 产品。它整理的是构建 DeepSeek-native Agent 时最容易被忽略、但会直接影响成本、稳定性和工具调用正确性的工程模式。

## 为什么需要这个项目

很多 Agent 框架已经可以“接入 DeepSeek API”，但接入不等于用好。DeepSeek V4 的关键能力包括：

- Thinking mode 默认开启，并通过 `reasoning_content` 返回思考内容。
- Thinking + tool calls 场景中，发生工具调用的 assistant 消息必须在后续请求里保留 `reasoning_content`。
- Context caching 默认开启，缓存命中依赖稳定的前缀复用。
- API 返回 `prompt_cache_hit_tokens` 和 `prompt_cache_miss_tokens`，可以直接驱动上下文编排。
- V4 Flash / V4 Pro 适合做成本、延迟和推理深度路由。

本项目关注的是：如何把这些能力变成 Agent 架构，而不是只把模型名填进 provider 配置。

## 适合谁读

- 正在用 DeepSeek API 构建 coding agent、desktop agent 或 workflow agent 的开发者。
- 已经有 OpenAI-compatible Agent，但想更好利用 DeepSeek thinking / cache / tool calls 的团队。
- 想研究 Agent 上下文管理、工具安全、文件编辑回滚机制的人。

## 核心模式

### 1. DeepSeek-native LLM Client

不要只做 OpenAI-compatible 薄封装。DeepSeek-native client 应该显式建模：

- `thinking` 开关：enabled / disabled
- `reasoning_effort`：high / max
- `reasoning_content`：流式解析、保存、按需回传
- tool calls：分片参数拼接、多轮 tool call 链路
- cache metrics：hit / miss token 读取
- DeepSeek-specific 错误分类：认证、余额、限流、上下文超限、内容过滤、参数不兼容

### 2. Reasoning Content Chain

普通多轮对话里，旧的 `reasoning_content` 可以不参与后续上下文。但如果该 assistant 消息触发了 tool call，它的 `reasoning_content` 必须随 tool call 历史一起回传。

推荐规则：

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

这不是“展示思考过程”的 UI 问题，而是协议正确性问题。

### 3. Cache-friendly Prompt Boundary

DeepSeek context cache 对稳定前缀敏感。Agent 的提示词应拆成：

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

原则：稳定内容尽量靠前，动态内容尽量靠后。不要把每轮变化的任务状态插进 system prompt 中间，否则会破坏缓存前缀。

### 4. Context Budget for 1M-token Agents

1M context 不是垃圾桶。上下文越大，越需要预算。

一个可用的默认预算：

| Segment | Ratio | Purpose |
|---|---:|---|
| system | 15% | identity, behavior, safety |
| tool definitions | 5% | stable tool schemas |
| history | 50% | conversation and tool results |
| current task | 10% | active request and working set |
| reserve | 20% | output and emergency headroom |

压缩顺序：

1. 删除旧的非工具调用 reasoning 内容。
2. 截断长工具输出，保留头部和尾部结构。
3. 折叠旧 tool call / tool result 配对。
4. 摘要旧对话轮次。
5. 最后才丢弃原文。

### 5. Tool Safety Attributes

不要让 LLM 临时判断工具是否安全。工具安全属性应该是工具代码的一部分：

```python
ToolSafety(
    is_readonly=True,
    is_destructive=False,
    needs_confirmation=False,
    is_concurrency_safe=True,
    risk_level=1,
)
```

这些属性可以驱动：

- Plan mode 物理移除写工具。
- 读工具并行、写工具串行。
- 高风险工具调用前确认。
- 工具列表和 schema 的安全过滤。
- 审计日志和风险提示。

核心思想：安全边界应尽量由确定性控制面实现，而不是靠提示词劝模型乖一点。

### 6. File Editing Trust Layer

Coding agent 的文件编辑不应该只有“写入成功”。最低信任层应包含：

- 编辑前 checkpoint。
- diff preview。
- 确认后 hash 校验，防止确认期间文件被外部修改。
- patch-first editing。
- timeline。
- rewind。

推荐文件编辑流程：

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

建议把模型路由做成规则驱动，而不是再问一次模型。

示例：

| Task | Model | Thinking |
|---|---|---|
| short chat / simple command | V4 Flash | off |
| normal agent task | V4 Flash | on |
| multi-tool / debugging / analysis | V4 Flash | deep / max |
| large repo analysis / high-stakes planning | V4 Pro | deep / max |

路由输入应包括：

- 用户请求长度和关键词。
- 当前上下文估算 token。
- 最近 cache hit ratio。
- 工具调用轮数。
- 失败重试次数。

### 8. From Chat Loop to Recoverable Tasks

一个可托付的 Agent 不只是 while-loop。建议引入任务生命周期：

```text
task:
  pending -> running -> paused -> completed
                  |          |
                  v          v
                failed    resumed
```

每个任务保存：

- user input
- current step
- model / thinking mode
- messages snapshot
- last checkpoint id
- token and cost usage
- failure reason

这样 Agent 才能处理中断、恢复、长任务和用户审计。

## 推荐发布形态

这个项目适合作为 pattern library 发布：

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

## 不是这个项目要做的事

- 不提供完整桌面 Agent 产品。
- 不收录泄漏源码、完整提示词或非公开资料。
- 不声称任何第三方 Agent 内部实现。
- 不提供生产级沙箱。

## 参考资料

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
