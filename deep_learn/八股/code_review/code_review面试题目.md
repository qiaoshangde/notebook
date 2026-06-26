
我做过一个基于 Agentic Workflow 的自动化 Code Review Agent，主要面向 GitHub PR 审查场景。这个项目的目标不是简单调用一次大模型生成评论，而是模拟一个真实 reviewer 的工作流程：先判断 PR 是否适合自动处理，再采集 PR 的 diff、提交历史、文件内容和历史审查规则，然后由 LLM 分析安全漏洞、Bug、代码规范和性能问题，最后根据结果生成 Review 决策、inline comments，必要时还能创建自动修复 PR。

这个项目整体采用三层设计：最上层是 Skill 层，用 Markdown Prompt 定义审查流程、人工介入规则、自动修复边界和输出格式；中间是 Agent 层，负责多轮 LLM tool calling 主循环，包括调用模型、解析 tool call、执行工具、回填工具结果和维护上下文；最底层是 Tool 层，封装 GitHub API 和本地 Memory 能力，比如读取 PR、获取 diff、提交 review、创建分支、更新文件、创建 Fix PR，以及维护开发者画像和仓库问题统计。

在执行流程上，我把一次 PR 审查拆成四个阶段。第一阶段是 Triage，判断 PR 是否适合自动审查，比如 PR 是否过大、是否是 Draft、是否包含 Breaking Change、是否涉及多个安全核心文件；如果风险过高，就降级为人工审查。第二阶段是 Analyze，采集上下文，包括 PR 基本信息、变更文件、diff、提交历史、完整文件内容和 Memory 中的已知问题模式、误报规则、团队代码规范。第三阶段是 Review，由 LLM 根据这些上下文识别 SQL 注入、硬编码密钥、XSS、空值错误、边界条件问题、性能问题等，并输出结构化的 issues 和最终 decision。第四阶段是 Act，把审查结果落到 GitHub 上，比如提交 review comments、创建修复分支、写入修复文件、创建 Fix PR，并更新 Memory。

这个项目里我封装了 11 个 GitHub 工具和 6 个 Memory 工具。GitHub 工具包括获取 PR 信息、获取 changed files、读取文件内容、提交 review、创建分支、更新文件、创建 PR、添加 comment、检测测试框架等。Memory 工具主要用于长期经验沉淀，包括读取已知审查规则、添加问题模式、添加误报规则、获取和更新开发者画像、聚合仓库级问题模式。这样 Agent 不只是看当前 PR，而是可以跨 PR 积累经验，比如某个开发者经常出现安全问题，或者某个仓库反复出现某类 high severity bug。

我还实现了 dry-run 模式，这是这个项目里比较重要的工程安全设计。因为自动 Code Review 涉及很多写操作，比如提交 PR Review、创建分支、写文件和创建 Fix PR。如果开发调试时直接执行这些操作，很容易污染真实仓库。所以我在 ToolRouter 层区分了读工具和写工具：读操作会真实执行，以便拿到真实 PR 数据；写操作在 dry-run 下会被拦截，只返回模拟结果。这样既能跑通完整 Agent 流程，又不会真的修改 GitHub。

另一个重点是 trajectory 轨迹采集。每次 Agent 执行时，系统会把用户输入、LLM 响应、tool use、tool result、token 消耗、最终 decision 和 issues_found 都记录成 JSONL 文件。这个设计有两个价值：第一是方便调试，因为 Agent 出错时不能只看最终答案，需要复盘它中间调用了哪些工具、工具返回了什么；第二是可以把这些轨迹导出成训练数据，比如 SFT 数据、工具监督数据和 preference pairs，为后续训练更专用的 Code Review Agent 做准备。

在数据导出方面，我特别处理了工具调用格式。早期如果把工具调用直接序列化成纯文本，比如 `[Tool: get_pull_request]`，模型学到的只是文本格式，不是真正的 function calling。这个项目里导出 SFT 数据时会保留 OpenAI native tool_calls 结构，也就是 assistant 消息中有 `tool_calls` 字段，工具返回用 `role=tool` 和 `tool_call_id` 对应。这样训练数据和真实部署时的工具调用格式更一致。

项目里我也考虑了上下文窗口限制的问题。PR diff 可能很大，如果全部塞给 LLM，会导致 token 成本高、响应慢，也容易出现 lost in the middle。因此 ToolRouter 对大 patch 和完整文件内容做了截断，同时在 Triage 和 Analyze 阶段优先关注高风险文件，比如 auth、security、permission、token 相关文件。如果需要进一步优化，我会把固定字符截断升级成风险感知的上下文选择，比如按 diff hunk 分块、按文件风险排序，或者结合静态分析工具做预筛选。

这个项目目前更偏 Agent 原型和工程验证，不是生产级系统。我认为它的亮点是把 LLM、工具调用、GitHub API、Memory 和 trajectory 数据闭环串了起来，而不是只做一次 prompt 调用。它也有一些可以继续改进的地方，比如现在流程对 Prompt 依赖比较强，后续可以把 Triage、Analyze、Review、Act 改成显式状态机；LLM 输出 JSON 也可以加更严格的 schema 校验和重试机制；自动修复 PR 也应该加测试运行、diff 校验和人工确认，避免模型生成不安全的修复。

整体来说，这个项目让我更深入理解了 Agent 工程化的几个核心问题：一是如何把模型能力和外部工具结合起来；二是如何控制工具调用的安全边界；三是如何记录 Agent 执行过程用于调试和训练数据沉淀；四是如何在长上下文、工具调用和真实 API 副作用之间做工程权衡。