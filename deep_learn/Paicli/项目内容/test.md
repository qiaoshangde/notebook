## 10 个顶层接口

|接口|所属模块|主要作用|主要实现|
|---|---|---|---|
|[LlmClient.java (line 9)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\llm\\LlmClient.java:9)|模型层|统一模型对话、工具调用、流式响应和模型能力|`AbstractOpenAiCompatibleClient`，再由 GLM、DeepSeek、Kimi 等继承|
|[Memory.java (line 13)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\memory\\Memory.java:13)|记忆层|统一记忆的保存、查询、搜索、删除和 Token 统计|`ConversationMemory`、`LongTermMemory`|
|[Renderer.java (line 23)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\render\\Renderer.java:23)|渲染层|统一终端、TUI、inline、微信等输出方式|`PlainRenderer`、`InlineRenderer`、`LanternaRenderer`、微信 Renderer|
|[HitlHandler.java (line 14)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\hitl\\HitlHandler.java:14)|人工审批|统一向用户发起审批并获取审批决定|`TerminalHitlHandler`、`RendererHitlHandler`、`TuiHitlHandler`|
|[McpTransport.java (line 9)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\mcp\\transport\\McpTransport.java:9)|MCP|屏蔽 MCP 底层通信方式的差异|`StdioTransport`、`StreamableHttpTransport`|
|[SearchProvider.java (line 16)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\web\\SearchProvider.java:16)|联网搜索|统一不同搜索服务的调用方式|`ZhipuSearchProvider`、`SerpApiSearchProvider`、`SearxngSearchProvider`|
|[CodeSearchEngine.java (line 6)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\tool\\CodeSearchEngine.java:6)|代码定位|统一 ripgrep 与纯 Java 搜索|`RipgrepCodeSearchEngine`、`JavaCodeSearchEngine`|
|[BrowserConnector.java (line 3)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\browser\\BrowserConnector.java:3)|浏览器|统一浏览器连接、断开和状态查询|`Main` 中创建匿名实现，注入 `ToolRegistry`|
|[TaskRunner.java (line 4)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\runtime\\task\\TaskRunner.java:4)|Runtime API|把一句 prompt 转换成一次 Agent 任务执行|主要通过 Lambda 或方法引用提供|
|[WechatMessageSender.java (line 6)](E:\\idea_project\\paicli-main\\src\\main\\java\\com\\paicli\\wechat\\WechatMessageSender.java:6)|微信通道|屏蔽微信消息实际发送方式|主要通过 Lambda/连接对象注入|

## 推荐阅读顺序

建议先看最影响主执行链路的五个：

```
1. LlmClient
2. Memory
3. Renderer
4. HitlHandler
5. McpTransport
```

然后再看外围能力：

```
6. SearchProvider
7. CodeSearchEngine
8. BrowserConnector
9. TaskRunner
10. WechatMessageSender
```

嵌套接口不需要专门集中学习，读到对应模块时顺带理解即可。

## 有两个容易误解的地方

第一，`MemoryManager` 没有实现 `Memory`。

```
Memory
├── ConversationMemory
└── LongTermMemory

MemoryManager
└── 协调上面两种 Memory、检索器、压缩器和 TokenBudget
```

所以看懂 `Memory` 接口，只能理解记忆容器的统一能力，还不能看懂完整记忆系统。

第二，工具模块没有一个顶层 `Tool` 接口。工具描述使用的是 `LlmClient.Tool` 数据结构，真正执行动作通过 `ToolRegistry.ToolExecutor` 嵌套接口完成。



