
FirstCoder每次给主模型发送的请求，可以分成 `messages`、工具定义和生成参数三部分。先说 `messages`，它是严格按顺序构建的。
> 
> 第一部分是系统前缀。最前面是一条稳定的 system message，内部依次包含基础规则、Agent工作规范、项目的 `AGENTS.md`、Skill协议与可用 Skill、Provider能力，以及权限策略。接着根据当前运行状态，可能再追加临时运行指令、任务验收条件和当前 TaskPlan，这些也都是 system message。
> 
> 第二部分是压缩后的历史。如果存在 Checkpoint，就先放一条 Checkpoint摘要，再从 Checkpoint指定的消息边界继续；如果没有 Checkpoint，就使用当前全部有效消息。如果之前有内容被裁剪，还会放一个“早期对话已裁剪”的提示。
> 
> 第三部分是真实消息尾部，按照发生顺序放入用户消息、模型回复、工具调用和工具结果。用户消息会带真实 message ID，供任务边界判断使用；工具调用后必须紧跟对应的工具结果，发送前还会校验这个协议。第一次调用通常以当前用户消息结尾，工具执行后的下一次调用通常以工具结果结尾。
> 
> `messages` 之外，请求还会携带当前可用工具的 Schema、`tool_choice`、温度、最大输出 Token 和 Provider私有参数。最后由不同 Provider适配器补充模型名称并转换成 OpenAI或Anthropic需要的格式。

### 请求的完整顺序

```
ChatRequest
│
├── messages
│   │
│   ├── 1. 稳定系统提示词
│   │      ├── 基础规则
│   │      ├── Agent工作规范
│   │      ├── AGENTS.md
│   │      ├── Skill协议
│   │      ├── 可用Skill
│   │      ├── Provider能力
│   │      └── 权限策略
│   │
│   ├── 2. 临时运行指令                  可选
│   ├── 3. 验收条件                      可选
│   ├── 4. 当前TaskPlan                  可选
│   ├── 5. Checkpoint摘要                可选
│   ├── 6. 早期对话已裁剪标记             可选
│   │
│   └── 7. 真实消息tail
│          ├── user
│          ├── assistant文本或tool_call
│          ├── tool_result
│          └── 后续user/assistant/tool...
│
├── tools：当前可见的工具Schema
├── tool_choice：auto、none、required或指定工具
├── temperature
├── max_tokens
└── extra_body：Provider私有参数
```

#### 1. 稳定系统提示词

系统提示词内部顺序由 [prefix.py](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\prompting\\domain\\prefix.py) 决定。

这些内容会合并成第一条 system message，并通过指纹缓存；项目规则或Provider能力变化时才重新生成。

#### 2～4. 动态system消息

动态内容依次追加：

```
runtime_instruction
→ acceptance_contract
→ task_plan
```

对应 [request.py](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\agent\\preparation\\request.py)。

其中：

- `runtime_instruction`：本轮临时指令，例如结束前要求协调未完成任务。
- `acceptance_contract`：当前任务明确的验证和验收要求。
- `task_plan`：最新结构化任务计划，只发送当前快照。

#### 5～7. Checkpoint与真实历史

顺序是：

```
Checkpoint summary
→ trimmed marker
→ checkpoint之后的真实tail
```

没有 Checkpoint 时，直接从当前有效消息开始。对应 [builder.py](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\context\\domain\\projection\\builder.py)。

发送前还会：

- 去掉内部 `system_meta`。
- 忽略已经完全裁剪的空消息。
- 把用户图片转换成多模态内容。
- 把 assistant内部的工具调用转换成 Provider工具调用。
- 把工具结果转换成 `role=tool`。
- 校验工具调用和结果必须配对。

#### 顶层请求字段

最终的中立请求结构是：

```
ChatRequest(
    messages=...,
    tools=...,
    tool_choice=...,
    temperature=...,
    max_tokens=...,
    extra_body=...,
)
```

定义位于 [contracts.py](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\providers\\contracts.py)。

需要补充一句：

> 上述结构描述的是主 Agent请求。任务边界分类和 L4摘要也会调用模型，但它们属于内部辅助请求，使用专门的精简Prompt，并不会携带完整的主Agent上下文。

可能的追问：

1. 为什么工具Schema放在独立的 `tools` 字段，而不是system prompt中？
2. 没有Checkpoint和存在Checkpoint时，请求消息数量分别如何确定？
3. FirstCoder如何保证Checkpoint不会切断工具调用事务？
4. 上下文超过Token预算时，请求发送前会经历什么流程？
5. 为什么TaskPlan作为system消息，而不是普通user消息？
6. 第一次模型调用和工具执行后的第二次调用有什么区别？
7. OpenAI与Anthropic的system prompt和工具结果格式如何适配？
8. 如何记录哪些工具结果已经真正进入过模型上下文？

- 系统提示词为什么不能直接写进普通会话历史？
- FirstCoder如何决定一轮请求发送多少条历史消息？
- Checkpoint是如何减少上下文长度的？
- 如何保证截取历史后不会切断工具调用和工具结果？
- 为什么工具定义需要根据运行状态动态过滤？
- 工具结果发送给模型后，系统如何记录它已经被消费？
- 如果构建完成的请求仍然超过模型窗口，系统如何恢复？
- 主 Agent请求、任务边界请求和L4摘要请求有什么区别？
