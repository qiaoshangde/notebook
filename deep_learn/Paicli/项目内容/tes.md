
第二套 `conversationHistory` 压缩主要涉及以下文件。最核心的是前四个。

## 核心实现

| 文件                                                                                                                                                       | 作用                                       |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| [ConversationHistoryCompactor.java (line 32)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\memory\\ConversationHistoryCompactor.java:32) | 压缩算法本体：切分历史、调用 LLM 摘要、重建消息               |
| [Agent.java (line 159)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\agent\\Agent.java:159)                                              | ReAct 主循环调用压缩器，并维护 `conversationHistory` |
| [TokenBudget.java (line 113)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\memory\\TokenBudget.java:113)                                 | 估算消息、工具参数和图片占用的 Token                    |
| [ContextProfile.java (line 64)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\context\\ContextProfile.java:64)                            | 根据模型窗口计算自动压缩阈值                           |

关系如下：

```
ContextProfile
计算压缩阈值
        │
        ▼
Agent 调用前检查
        │
        ├── TokenBudget 估算 conversationHistory Token
        │
        ▼
ConversationHistoryCompactor
        │
        ├── 切分早期消息
        ├── 调用 LLM 生成摘要
        └── 重建 conversationHistory
```

## 三条执行路径

### 1. ReAct

[Agent.java (line 49)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\agent\\Agent.java:49)

主要关注：

- `conversationHistory` 字段；
- 构造 `ConversationHistoryCompactor`；
- 用户、助手和工具消息如何加入历史；
- 每次 LLM 调用前执行自动压缩；
- 手动压缩入口。

关键位置：

```
Agent.java:49    conversationHistory
Agent.java:69    创建 historyCompactor
Agent.java:141   加入 user 消息
Agent.java:159   LLM 调用前检查压缩
Agent.java:204   加入 assistant tool_call
Agent.java:219   加入 tool_result
Agent.java:230   加入最终 assistant 回答
Agent.java:279   手动 compactHistoryNow()
Agent.java:323   自动 maybeCompactHistory()
```

### 2. Plan

[PlanExecuteAgent.java (line 186)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\agent\\PlanExecuteAgent.java:186)

主要位置：

```
PlanExecuteAgent.java:147  创建压缩器
PlanExecuteAgent.java:186  自动压缩方法
PlanExecuteAgent.java:487  调用 LLM 前执行压缩
```

### 3. Multi-Agent

每个子 Agent 自己维护一份消息历史：

[SubAgent.java (line 101)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\agent\\SubAgent.java:101)

主要位置：

```
SubAgent.java:49   子 Agent 的 conversationHistory
SubAgent.java:63   创建压缩器
SubAgent.java:101  自动压缩方法
SubAgent.java:204  调用 LLM 前执行压缩
```

`AgentOrchestrator` 本身不直接压缩，它创建和调度 `SubAgent`，由每个 `SubAgent` 压缩自己的历史。

## 消息与模型接口

[LlmClient.java](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\llm\\LlmClient.java)

它定义了压缩器处理的数据结构：

- `Message`
- `ToolCall`
- `ChatResponse`
- `chat()`

压缩器要读取：

```
Message.role
Message.content
Message.toolCalls
ToolCall.function.name
ToolCall.function.arguments
```

然后调用：

```
llmClient.chat(summaryMessages, null);
```

`null` 表示摘要请求不提供工具列表。

## 手动 `/compact` 命令

### 命令解析

[CliCommandParser.java (line 85)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\cli\\CliCommandParser.java:85)

负责把：

```
/compact
```

解析成：

```
CommandType.COMPACT
```

### 命令执行和界面反馈

[Main.java (line 428)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\cli\\Main.java:428)

调用：

```
reactAgent.compactHistoryNow();
```

然后显示：

```
压缩前 Token → 压缩后 Token
```

## 微信通道中的 `/compact`

微信通道复用了 ReAct Agent 的手动压缩能力：

|文件|作用|
|---|---|
|[WechatCommandParser.java (line 44)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\wechat\\WechatCommandParser.java:44)|解析微信 `/compact`|
|[WechatMessageLoop.java (line 174)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\wechat\\WechatMessageLoop.java:174)|接收命令并调用会话|
|[WechatAgentSession.java (line 112)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\wechat\\WechatAgentSession.java:112)|调用 `agent.compactHistoryNow()`|

这三个只属于外围命令接入，不是压缩算法核心。

## 核心测试

### 压缩算法测试

[ConversationHistoryCompactorTest.java (line 15)](E:\\idea_project\\paicli-main\\src\\test\\java\\com\\paicli\\memory\\ConversationHistoryCompactorTest.java:15)

覆盖：

- 未达到阈值时不压缩；
- 手动压缩跳过阈值；
- 保留最近轮次；
- 保护工具调用关系；
- 空摘要不修改历史；
- LLM 异常不破坏历史。

刚才实际运行结果：

```
Tests run: 7
Failures: 0
Errors: 0
Skipped: 0
```

### 阈值测试

[ContextProfileTest.java (line 23)](E:\\idea_project\\paicli-main\\src\\test\\java\\com\\paicli\\context\\ContextProfileTest.java:23)

验证：

```
200k 窗口 → 167k 触发
1M 窗口   → 967k 触发
128k 窗口 → 95k 触发
```

### 命令测试

- [CliCommandParserTest.java](E:\\idea_project\\paicli-main\\src\\test\\java\\com\\paicli\\cli\\CliCommandParserTest.java)：验证 `/compact` 解析。
- [MainInputNormalizationTest.java (line 190)](E:\\idea_project\\paicli-main\\src\\test\\java\\com\\paicli\\cli\\MainInputNormalizationTest.java:190)：验证帮助列表包含 `/compact`。
- [WechatCommandParserTest.java](E:\\idea_project\\paicli-main\\src\\test\\java\\com\\paicli\\wechat\\WechatCommandParserTest.java)：验证微信命令解析。

## 推荐阅读顺序

如果只想看懂第二套机制，按这个顺序即可：

```
1. ConversationHistoryCompactorTest
2. ConversationHistoryCompactor
3. TokenBudget
4. ContextProfile
5. Agent.run()
6. Agent.maybeCompactHistory()
7. Agent.compactHistoryNow()
8. Main 中的 /compact
9. PlanExecuteAgent 和 SubAgent 的接入
```

最少只看这四个文件，就能理解主体：

```
ConversationHistoryCompactorTest.java
ConversationHistoryCompactor.java
Agent.java
ContextProfile.java
```