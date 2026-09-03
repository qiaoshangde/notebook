# 介绍一下多agent怎么做的

FirstCoder 的多 Agent 本质上是一个“父 Agent 通过工具委托子 Agent”的主从结构，并不是多个 Agent 共享上下文、自由聊天的群体协作系统。父 Agent 仍然是唯一的决策者，它负责理解用户需求、拆分任务和汇总结果；当它认为某项工作适合独立处理时，就调用 `delegate` 工具，指定子 Agent 的角色、具体任务、父任务摘要和可能涉及的文件路径。整体链路是：

```
父 Agent
  ↓ 调用 delegate
创建全新的子 Session
  ↓
启动一个新的 AgentLoop
  ↓
子 Agent 独立调用模型和工具
  ↓
返回总结、证据、修改文件和风险
  ↓
包装成 delegate 的工具结果
  ↓
父 Agent 继续判断和执行
```

`delegate` 是一个提供给父模型的普通工具，入口在 [delegate.py](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\tools\\infrastructure\\delegation\\delegate.py)。父模型可以选择四种角色：`researcher` 负责只读地搜索和理解代码；`reviewer` 负责审查调用链、代码差异和风险；`tester` 可以在只读工具之外使用 Shell 和 Python 做验证；`coder` 还可以使用写文件、编辑、删除和补丁工具完成代码修改。不同角色不是使用不同的 Agent 类（FirstCoder 的父子 Agent 共用同一套 AgentLoop 实现，但每个 Agent 都拥有独立的 AgentLoop 实例和 Session；角色差异由提示词、工具白名单、执行预算和隔离策略决定。），而是使用同一个 AgentLoop（不是同一个实例，而是根据这个agentloop类，给不同的参数，创建不同的对象），通过不同的角色提示词、工具白名单和运行预算表现出不同能力，这些配置定义在 [contracts.py](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\delegation\\contracts.py)。

父 Agent 发起委托后，[SubagentRunner](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\delegation\\application\\subagent.py) 会创建一个全新的子 Session。这个子 Session 不会复制父会话的完整消息历史，只记录父 Session ID、父任务哈希、子 Agent 角色和委托任务等关联信息。子 Agent 真正能看到的是父 Agent显式传入的任务、摘要、路径提示，再加上项目的 AGENTS.md、系统规则和自己允许使用的工具。这种方式可以避免把父 Agent 已经积累的大量上下文全部复制给子 Agent，也能降低无关信息对它的干扰。

子 Session 创建完成后，会通过 [AgentLoopChildExecutor](E:\\agent\\后端agent开发资料\\FirstCoder\\firstcoder\\agent\\adapters\\delegation.py) 再启动一个普通 AgentLoop。也就是说，子 Agent 不是一个简单的单次模型调用，它内部也可以经历“模型推理—调用工具—读取结果—继续推理”的完整循环。但它没有后台任务管理器，也不会注册 `delegate` 工具，因此不能继续创建孙 Agent，避免无限递归委托。子 Agent 如果执行到需要用户交互的权限确认，也无法直接询问用户，而是作为本次委托失败返回给父 Agent。

委托支持前台和后台两种方式。前台委托时，父 Agent 会等待子 Agent 完整执行结束，然后拿到最终总结，再继续自己的工具循环。后台委托时，使用的就是前面讲过的通用后台工具机制：父 Agent 立即获得后台任务 ID，子 Agent 在后台线程中执行，完成后再通过 `<task_notification>` 通知父 Agent。因此多个 researcher、reviewer 或 tester 可以作为多个后台任务并发运行，但它们之间没有直接通信，结果最终都由父 Agent收集和整合。

`coder` 角色的后台执行更特殊，因为它具有修改文件的能力。如果让后台 coder 直接操作父工作区，可能和父 Agent 或其他 coder 同时修改相同文件。因此 FirstCoder 要求后台 coder 必须运行在独立的 Git Worktree 中，每个 coder 使用单独的目录和分支，并重新创建以该 Worktree 为根目录的工具和权限管理器。子 Agent 完成后，系统只把修改文件列表、分支、Worktree 路径和 diff 摘要返回给父 Agent，不会自动合并到父工作区。这样父 Agent可以先审查结果，再决定如何处理；如果当前目录不是 Git 仓库，后台 coder 会被直接拒绝。前台 coder 则可以直接在父工作区执行，仍然接受正常的权限控制。

每类子 Agent 还有独立的调用次数和时间上限。例如 reviewer 的预算比较小，coder 和 tester 的预算更大，用来避免一次委托无限消耗模型调用和工具轮次。子 Agent 最终需要返回简洁报告，包括工作结论、证据、修改文件和潜在风险；父 Agent只接收这份结果，不会自动继承子 Agent 的完整思考过程和会话历史。

因此，FirstCoder 的多 Agent 可以概括为：父 Agent通过 `delegate` 工具按角色创建拥有独立 Session 和独立上下文的子 Agent，子 Agent使用受限工具完成一项明确任务，再把结构化结果返回父 Agent；多个子 Agent可以借助后台任务并发执行，具有写能力的后台 coder 则通过 Git Worktree 隔离修改。它属于受控的层级式任务委托，而不是完全自治、共享记忆的多 Agent 网络。

# 多agent场景下 你是怎么做好子agent的拆分的 硬约束？



# 有设计兜底机制吗 怎么解决这些子agent上下文污染的问题
FirstCoder 解决子 Agent 上下文污染的核心思路，不是在一份共享上下文里做过滤，而是直接让每个子 Agent 使用全新的 Session 和独立的 AgentLoop 实例。父 Agent调用 `delegate` 后，系统不会把父会话的全部用户消息、模型回答、工具调用和压缩历史复制给子 Agent，而是只传递明确的委托信息，包括任务描述、父 Agent主动整理的摘要、路径提示、角色说明和项目根目录。子 Agent拥有自己的 JSONL 会话日志、SessionView、上下文压缩状态和工具调用历史，因此 researcher 的搜索过程不会进入 tester 的上下文，多个子 Agent 之间也不会互相继承消息。

子 Agent 的上下文大致是这样构建的：

```
子 Agent 自己的系统提示
+ 项目的 AGENTS.md
+ 当前角色及职责
+ 父 Agent 提供的摘要
+ 路径提示
+ 明确的委托任务
+ 子 Agent 自己后续产生的消息和工具结果
```

而不是：

```
父 Agent 的完整历史
+ 其他子 Agent 的完整历史
+ 当前委托任务
```

角色限制也在降低污染。researcher 只获得代码搜索和读取工具，reviewer 只关注审查，tester 才能运行验证命令，coder 才能修改文件。子 Agent还拿不到 `delegate` 工具，不能继续创建孙 Agent。这样可以让每个子 Agent围绕一个明确目标工作，避免无关工具、无关任务和递归委托不断扩大上下文。

子 Agent完成后，它的完整执行历史仍然保留在自己的子 Session 中，不会自动合并回父 Session。父 Agent只会收到子 Agent最终输出的总结，它被包装成一次 `delegate` 工具结果。也就是说，父 Agent看到的是结论、证据、修改文件和风险，而不是子 Agent每一轮推理和所有工具输出。这相当于在父子 Agent之间建立了一个受控的上下文接口：

```
父 Agent → 精简任务说明 → 子 Agent
子 Agent → 最终结果摘要 → 父 Agent
```

还需要区分“模型上下文污染”和“工作区状态相互影响”。独立 Session 解决的是消息、记忆和工具历史污染；如果子 Agent直接读取同一个项目目录，它仍然能看到父 Agent已经修改的文件，这是共享环境，不属于模型上下文共享。对于能够修改代码的后台 coder，FirstCoder 进一步创建独立 Git Worktree，让它连文件环境也和父 Agent隔离，完成后只返回分支、Worktree 路径和 diff 摘要，不自动合并修改。

不过当前实现只是降低污染，并不能完全消除污染。因为子 Agent仍然继承项目的 AGENTS.md，而且父 Agent传入的 `parent_summary` 如果包含错误、无关信息或者误导性判断，子 Agent仍可能受到影响；子 Agent最终报告主要依靠提示词要求保持简洁，并没有非常严格的结构化结果校验。因此更准确地说，FirstCoder 通过“独立 Session、最小化上下文传递、角色工具白名单、禁止嵌套委托、只返回最终摘要和可写 Agent 的 Worktree 隔离”来控制上下文污染，而不是让所有 Agent共享同一份会话后再做清理。


# 子agent和普通的工具执行过程是一样的是吗？
对，从父 Agent 的视角看，`delegate` 和普通工具的执行过程基本一样：模型生成工具调用，工具执行层进行参数校验和权限检查，然后调用工具执行器，最后把 `ToolResult` 写入 Session，再请求模型继续处理。区别只是普通工具的执行器通常直接读文件、运行命令或者修改代码，而 `delegate` 的执行器内部会创建一个新的子 Session 和子 AgentLoop，让子 Agent完成一整轮“模型调用—工具执行”循环，最后把子 Agent的总结包装成 `delegate` 的工具结果返回父 Agent。

可以理解成两层嵌套：

```
父 AgentLoop
    ↓ 调用普通工具 delegate
delegate 工具执行器
    ↓ 启动
子 AgentLoop
    ↓ 多轮调用模型和工具
返回子 Agent最终总结
    ↓
包装成 delegate 的 ToolResult
    ↓
父 AgentLoop 继续
```

如果是前台调用，父 Agent会像等待普通工具一样等待 `delegate` 返回；如果设置了 `run_in_background=true`，它也和其他后台工具一样，由通用后台任务机制提交线程池，先返回任务 ID，子 Agent完成后再发送后台通知。所以父层的工具协议和执行框架是一样的，特殊之处只在于 `delegate` 工具内部执行的不是一个简单函数，而是另一个完整的 AgentLoop。





# 进程和线程的区别


进程是操作系统进行资源分配和隔离的基本单位，每个进程通常拥有独立的虚拟地址空间、文件描述符和运行环境；线程则是进程内部的执行单元，同一个进程中的线程共享内存、代码和大部分资源，但每个线程拥有自己的调用栈、寄存器和执行位置。因为线程共享数据，所以创建和切换成本通常比进程低，线程之间传递数据也更方便，但同时需要使用锁来避免数据竞争，一个线程发生严重错误也可能影响整个进程。进程之间隔离性更强，一个进程崩溃通常不会直接破坏其他进程，但进程创建、上下文切换和进程间通信的成本更高。在 Python 的 CPython 实现中，还需要考虑 GIL：多个线程适合网络请求、文件读写、Shell 等 I/O 密集型任务，因为线程等待 I/O 时可以让其他线程运行；对于大量 Python 计算的 CPU 密集型任务，多线程通常不能充分利用多核，更适合使用多进程。FirstCoder 的后台工具主要是文件、网络、命令和子 Agent调用，因此选择线程池能够以较低成本共享后台任务管理器和执行环境；如果未来要执行高强度计算、需要强制终止任务或者希望获得更强的故障隔离，就更适合改用独立进程或外部 Worker。

可能的追问包括：什么是 GIL；线程为什么需要加锁；进程间如何通信；协程和线程有什么区别；线程池与进程池如何选择；为什么 Python 线程仍然适合 I/O 密集型任务；一个线程崩溃是否一定会导致整个进程退出。

