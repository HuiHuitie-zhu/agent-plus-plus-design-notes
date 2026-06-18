# 发布说明草稿

## GitHub 重复度判断

截至 2026-06-18，公开资料里常见的是三类项目：

1. 支持 DeepSeek API 的 coding agent 或 IDE 插件，例如 Aider、Roo Code、Continue。
2. 通用 Agent 框架，例如 OpenHands。
3. 学术或实验型上下文压缩、prompt caching、memory folding、sandbox checkpoint 研究。

没有明显看到一个直接定位为“DeepSeek-native Agent 工程模式库”的同质项目。

因此建议定位为：

> A practical pattern library for building DeepSeek-native agents.

不要定位为：

> Another desktop AI agent.

## 差异化卖点

- 不是“支持 DeepSeek provider”，而是解释如何利用 DeepSeek API 语义。
- 关注 thinking + tool calls 的 `reasoning_content` 链路。
- 关注 context cache 命中率与提示词边界。
- 关注 1M context 下的预算和压缩顺序。
- 关注工具安全属性和文件编辑信任层。

## 风险清单

### 高风险：非公开资料与泄漏来源

不要发布：

- 泄漏源码原文。
- 完整系统提示词。
- 工具定义逐字复刻。
- 对第三方内部实现的确定性断言。

推荐改写：

- “Inspired by public agent engineering patterns”
- “Based on public documentation and implementation experience”
- “A practical pattern library”

### 中风险：误导为生产级 Agent

需要在 README 顶部明确：

- documentation-first
- research prototype
- not production sandbox
- no warranty

### 高风险：隐私与密钥

发布前删除或加入 `.gitignore`：

```gitignore
.env
data/
*.log
__pycache__/
.pytest_cache/
frontend/node_modules/
frontend/dist/
```

### 中风险：安全工具误用

如果放示例代码，默认应使用只读模式。写文件、bash、macOS 控制能力应拆成单独示例，并写明风险。

## 推荐仓库描述

中文：

> DeepSeek-native Agent 工程模式库：thinking、tool calls、context cache、上下文预算、工具安全与文件编辑信任层。

English:

> Engineering patterns for DeepSeek-native agents: thinking, tool calls, context caching, context budgets, tool safety, and file-edit trust layers.

## 推荐 Topic

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

## 首发内容建议

第一版只发 README，不急着搬完整代码。

后续再补：

1. minimal DeepSeek client
2. context budget module
3. tool safety schema
4. file checkpoint example
5. task lifecycle example
