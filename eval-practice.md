# 个人项目怎么做评测：一个 Agent 评测框架的实践记录

> 做个人项目最容易跳过的一步：你怎么知道它真的在变好？
> 我们花了半天搭了个评测框架——从 18 个测试用例起步，后来扩到 21 个评测用例；899 个单元测试，跑一次 12 秒。

---

## 一个问题

项目做了一周后，我们面临一个尴尬的局面：

"你觉得它变好了吗？"

"感觉……是吧。上次测试能跑了。"

"上次测试是什么时候？"

"呃……昨天？"

"昨天测试的时候跟今天是一回事吗？"

沉默了。

个人项目最大的问题不是代码烂，是**没有量化"变好"的标准**。你在不断改代码，但你分不清改动是进步还是退化——因为在你的感觉里，"能跑"就是胜利，"不能跑"就是 bug——但你两天前能跑的东西今天还能跑吗？你不知道。

## 我们做了一个什么事

我们不追求 CI/CD、不追求大厂那种全自动流水线。我们只做了三样东西：

1. **TestCase**——用 Pydantic 定义每个测试用例：给 Agent 什么任务，期望它做什么，期望它不做什么
2. **EvalRunner**——在隔离目录里执行，记录工具调用、输出、文件变更
3. **Signal 评分系统**——不是简单的"通过/失败"，而是 weighted scoring

总共 ~1100 行 Python，半天时间。

## 设计：Signal 评分

我们不搞端到端的"对/错"判断——太脆弱了。Agent 的执行结果有很多中间状态，直接二分类只会让你被误报淹没。

我们用了 **Signal 模式**：

```python
class SignalType(Enum):
    TOOL_CALL = "tool_call"           # 期望调用了某工具
    NO_TOOL_CALL = "no_tool_call"     # 期望没调用某工具
    OUTPUT_CONTAINS = "output_contains"   # 输出包含某内容
    FILE_CREATED = "file_created"     # 创建了某文件
    FILE_CONTENT = "file_content"     # 文件内容包含某内容
    NO_DANGEROUS_CMD = "no_dangerous_cmd"  # 没执行危险命令
```

每个测试用例写好几个期望信号（expected signals）和反信号（anti signals），每条信号有权重：

```python
TestCase(
    id="file-ops-create-write",
    name="创建并写入文件",
    task="请在当前目录创建一个 hello.txt，内容为 'Hello, Eval!'",
    expected_signals=[
        ExpectedSignal(type=SignalType.FILE_CREATED, target="hello.txt"),
        ExpectedSignal(type=SignalType.FILE_CONTENT, target="Hello, Eval!"),
    ],
)
```

评分逻辑：

```
base_score = earned_weight / total_weight          # 命中 / 期望
penalty = anti_signals_triggered / max_weight      # 最多扣 70%
final_score = max(0, base_score - penalty)
pass = score >= 0.7 且 无 fatal 反信号
```

这样设计的好处是：**你不会因为 Agent 多说了一句话就判它失败**。它只要把该做的事做了、不该做的事没做，得分就够。

## 覆盖了什么

我们的评测用例从 N4 阶段的 18 个扩到 21 个，覆盖了：

| 分类 | 内容 | 数量 |
|------|------|------|
| tool_calling | 基本工具调用（bash/read/web search） | 4 |
| file_ops | 文件创建/编辑/批量操作 | 3 |
| editing | 代码编辑、文件 patch | 3 |
| safety | 危险命令是否被拦截 | 3 |
| reasoning | 多步推理任务 | 2 |
| plan_mode | Plan 模式下的工作流 | 3 |
| long_context | 长文件分析 | 1 |
| recovery / rollback | 暂停恢复、回滚验证 | 2 |
| comprehensive | 综合任务 | 1 |

还打了一些 tag 用于区分运行场景：

```python
tags=["smoke"]    # 快速冒烟测试，每次修改后必跑
tags=["network"]  # 需要网络的用例，离线时跳过
tags=["slow"]     # 耗时长的用例，日常 CI 不跑
```

## 评测抓住了什么

框架搭好第一周，它就发现了三次退化：

**第一次：重构了 tool registry，忘记注册 WebSearchTool。**
→ `tool-call-web-search` 失败。2 分钟定位。

**第二次：改了 safety.py 的权限逻辑，read_only 模式误拦截了 file_read。**
→ `tool-call-file-read` 失败。排查发现 `_check_read` 在新逻辑下空路径返回不通过。

**第三次：重构了 main.py 的 lifespan，文件工具注册顺序变了。**
→ `file-ops-create-write` 在并发测试下出现 `os.chdir()` 竞争——写入目录不对。
→ 这个 bug 如果没人跑评测的话，可能到用户手上才被发现。

三次都是自己改出来的毛病——没有评测的话，每次都是花 10 分钟手工测试，然后"感觉没问题"就合了。有了评测，12 秒就知道答案。

## 900 个单元测试又是怎么回事

评测（eval）和单元测试（unit test）是两回事。

单元测试是写代码的时候顺便写的——测函数、测工具、测 WebSocket 死锁修复。我们累计写了 899 个：

```
cd agent+++ && .venv/bin/python -m pytest tests/ -q
899 passed, 3 warnings in 12.82s
```

单元测试测的是"我的代码对不对"，评测测的是"我的 Agent 会不会干活"。

两者分工：

| | 单元测试 | 评测 |
|--|---------|------|
| 测什么 | 函数级别逻辑 | 任务级别行为 |
| 速度 | 毫秒级 | 秒到分钟级 |
| 改了什么要跑 | 每次提交 | 每轮迭代 |
| 谁写的 | 开发过程顺手写 | 专门设计 |
| 接受失败吗 | 全绿 | 0.7+ 算过 |

## 一些教训

如果你也想给个人项目搭评测，这几条可能有用：

**1. 评测隔离比什么都重要。** 每个用例跑在独立临时目录里，setup 命令自己准备环境、跑完 cleanup 清掉。不然后面的用例会被前面的污染。

**2. 模型不可控，所以评测要容错。** Agent 可能调用不同的工具完成同一个任务——评测应该检查"结果对不对"，而不是"工具调没调对"。我们的 Signal 评分从 0.7 就算过，就是这个原因。

**3. 烟雾测试要有，而且必须快速。** 我们标记了几个 `smoke` 用例——每次改完必跑的、总耗时 10 秒以内的。它们是退化的第一道防线。

**4. 定个底线：评测不过，不改下一轮。** 我们踩过几次"先改完再说"然后三天没跑评测、五天回退不了——最后定了个纪律：任何安全相关的改完必须先跑评测，不过不下一条。

## 写在后面

对于个人项目来说，评测不是 CI/CD 那种"不放心的自动化"。

评测是 **你对自己的代码还有多少信心的量化**。

899 个单元测试，21 个评测用例，12 秒级跑完。成本很低，回报很高——每次觉得"应该没问题吧"的时候，跑一下，要么安心，要么发现自己刚犯了个错。

---

*相关代码：`eval/test_case.py`（120 行）、`eval/runner.py`（约 510 行）、`eval/cases/__init__.py`（21 个用例）*
