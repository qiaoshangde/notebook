

类的__init__时，会传入session信息，用的模型，工具注册表，工具执行器，上下文构造器；
上下文压缩管理器；任务边界分类器；后台任务管理器；子Agent；运行限制；遥测统计；停滞检测。

之后再处理一次请求时用的到。


run_user_turn和resume_with_user_input是两个对外的入口。其余的全是处理请求的内容函数。


首先看run_user_turn，这个函数，处理一次用户的请求（注意，是处理一次用户的请求，不是请求一次llm。）
这个函数的输入为用户的输入，附件（处理多模态信息），以及是否是流式调用

如果是流式就调用run_user_turn_streaming这个函数，
如果不是就调用_runuser_turn_sync这个函数






run_user_turn_streaming





run_user_turn_sync
非流式
这个函数，不是实现函数，而是包了一层try_expect。
调用的这个函数的实现（run_user_turn_sync_impl），当出现问题时，会调用persist_errored_turn处理问题。
在run_user_turn_sync_impl中，首先会检查是否还在等待，如果还在等待，就返回AgentTurnResult（status=AgentTurnStatus.WAITING_FOR_USER_INPUT）。

然后进行，检查附件（validate_attachments，检查是否正常以及模型是否支持视觉能力。）
然后，（使用begin_turn()函数）初始化这一轮Agent执行所需的临时状态；如果是在恢复权限确认，则尽量继续沿用原来那一轮的状态。



begin_turn()一个turn的台账
函数有一个参数，是否是新用户消息，如果是，就重新创建，如果不是，就说明是暂停的呢个turn（因为执行危险工具时，可能会暂停，请求用户意见。也就是说判断这个是一个新turn还是上次的turn。）
清空本轮激活的MCP工具
active_mcp_tool_names.clear()
重置Provider调用次数