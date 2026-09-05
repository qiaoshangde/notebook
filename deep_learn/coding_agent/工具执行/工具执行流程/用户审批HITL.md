
执行工具时，需要用户审批时，
因为工具的不同，在请求审批时，用户能看到的选项也不同

共有以下几种选项

1、deny
2、allow
3、allow always

其中支持写前预览的工具会出现下面一个选项
4、Apply reviewed change
以及输入框显示Reject ： 原因



# 什么是权限预检
FirstCoder 的权限预检，是在工具真正执行之前，由本地程序判断“这次操作能不能做”。模型生成工具名和参数后，系统会根据工具预先配置的 `ToolPermissionSpec`，提取操作类型、目标路径或命令、工作目录等信息，构造 `PermissionRequest`。权限管理器先匹配已有的长期授权规则，命中拒绝规则就返回 `DENY`，命中允许规则就返回 `ALLOW`；没有匹配时，再结合当前权限模式和具体操作参数进行判断。例如，项目内普通读取通常直接允许，敏感文件访问可能需要询问用户，而 Standard 和 Aggressive 下删除项目外文件会直接拒绝。最终，`ALLOW` 表示通过这一层权限判断，`ASK` 表示暂停并等待用户审批，`DENY` 表示不执行并返回失败结果。整个预检过程不会调用工具执行器，也不需要模型参与判断。另外，预检返回 `ALLOW` 后，文件修改仍可能进入写前审查，要求用户确认具体 diff，所以通过权限预检不代表一定马上执行。

可能的追问点：

1. 权限预检和写前审查有什么区别？
2. 为什么长期授权规则优先于权限模式？
3. `ASK` 后如何保存并恢复原工具调用？
4. 用户拒绝后，结果怎样返回模型？
5. 没有 `ToolPermissionSpec` 的工具如何处理？
6. 用户确认后，还需要检查哪些条件？




# ask时，给用户的选择有哪些
FirstCoder 的权限策略返回 `ASK` 后，普通权限确认通常会给用户三个选择：`Deny`、`Allow once` 和 `Allow always`。
`Deny` 只拒绝当前工具调用；
`Allow once` 只允许本次执行；
`Allow always` 除了允许本次执行，还会根据文件路径、命令前缀、网络主机或 MCP 工具等作用域保存一条长期授权规则，后续匹配的请求可以直接放行。不过，并不是所有工具都支持长期授权，例如 `python_exec` 和 `apply_patch` 会关闭 `allow_always`，此时只显示拒绝和允许一次。
如果这次 `ASK` 是由写前审查产生的，用户看到的是修改预览，主要选择是拒绝或者应用本次已预览的修改，不能保存长期授权；用户还可以在拒绝时附带反馈，让模型知道应该怎样调整代码。

可以归纳为：

```
普通权限确认：
1. Deny
2. Allow once
3. Allow always（仅工具允许时显示）

写前审查确认：
1. Deny
2. Apply reviewed change
3. Reject with feedback（通过文字反馈拒绝）
```

需要注意，当前没有“永远拒绝”的用户选项，`Deny` 和 `Reject with feedback` 都只针对本次调用。

可能的追问点：

1. 为什么部分工具不允许选择 `Allow always`？
2. `Allow always` 的作用域是怎样确定的？
3. `Deny` 和 `Reject with feedback` 有什么区别？
4. 为什么写前审查不能保存长期授权？
5. 用户允许后，系统如何保证执行的是原来的工具参数？
6. 用户长时间不确认，AgentLoop 会处于什么状态？
7. `Allow always` 生成的授权保存在哪里？


# 不同的工具ask时用户能看到什么

 用户在 `ASK` 时看到哪些选项，既取决于工具是否设置了 `allow_always=False`，也取决于这次暂停是“权限策略要求审批”，还是“写前审查要求确认”。

 一、支持 `Allow always` 的工具

下面这些工具没有关闭 `allow_always`，如果它们因为权限策略返回 `ASK`，用户通常会看到：

```
Deny
Allow once
Allow always
```

|工具类别|工具|
|---|---|
|文件读取|`ls`、`view`、`grep`、`glob`、`tree`、`read_multi`|
|文件修改|`write`、`edit`、`delete`|
|Shell 与进程|`shell`、`diagnostics`、`process_start`|
|Git|`git_status`、`git_diff`、`git_log`|
|网络|`fetch`、`web_search`|
|外部工具|动态加载的 MCP 工具|

其中，Git 的三个只读工具在项目内通常会被直接 `ALLOW`，所以正常情况下不会出现审批界面；只是它们的权限声明本身没有禁止 `Allow always`。

 二、不支持 `Allow always` 的工具

当前明确设置了 `allow_always=False` 的是：

```
python_exec
apply_patch
```

当它们进入普通权限审批时，用户只能看到：

```
Deny
Allow once
```

原因是它们每次调用的内容和影响范围变化很大，不适合通过一条简单 Scope 长期放行。

 三、同一个工具看到的选项也可能不同

以 `write` 为例，它本身支持 `Allow always`，但是在不同情况下，用户看到的选项可能不同。

 Standard 模式下写文件

Standard 默认认为写文件需要审批：

```
write("src/app.py")
→ 权限策略返回 ASK
→ 同时生成写前预览
```

此时用户看到：

```
Deny
Allow once
Allow always
```

还可以在拒绝时附带反馈。

 Aggressive 模式下写普通文件

Aggressive 的权限策略会自动允许项目内普通写入：

```
write("src/app.py")
→ 权限策略返回 ALLOW
```

但是普通交互式 Session 默认还开启写前审查，所以系统可能因为“需要用户确认这份 diff”而暂停。

这时候不是在询问是否长期授权，而是在询问是否应用当前预览，因此用户看到：

```
Deny
Apply reviewed change
```

不会显示 `Allow always`，因为权限策略本身已经允许，当前暂停只是写前审查门禁。

 Aggressive 模式下写敏感文件

例如：

```
write(".env")
```

Aggressive 也不会自动允许敏感写入：

```
权限策略返回 ASK
```

由于 `write` 支持长期授权，此时又会显示：

```
Deny
Allow once
Allow always
```

 四、四个修改工具的具体区别

| 工具            | 是否支持长期授权 | 普通权限 `ASK` 时                     | 仅写前审查暂停时                                                                  |
| ------------- | -------- | -------------------------------- | ------------------------------------------------------------------------- |
| `write`       | 是        | Deny / Allow once / Allow always | Deny / Apply reviewed change                                              |
| `edit`        | 是        | Deny / Allow once / Allow always | Deny / Apply reviewed change                                              |
| `delete`      | 是        | Deny / Allow once / Allow always | Deny / Apply reviewed change                                              |
| `apply_patch` | 否        | Deny / Allow once                | 理论上只有 Deny / Apply reviewed change，但当前它设置了 `allow_auto=False`，通常先进入普通权限审批 |

只要生成了写前预览，用户还可以使用：

```
Reject with feedback: <修改意见>
```

拒绝本次修改，并把原因返回给模型。




# Apply reviewed change什么时候会出现

`Apply reviewed change` 会在下面这种情况出现：

> 权限系统已经决定允许这次操作，但 Session 的写前审查要求用户确认实际修改内容。

完整条件是：

```
工具支持写前预览
并且
Session 开启了写前审查
并且
权限预检结果已经是 ALLOW
并且
当前不是 bypass 模式
```

支持写前预览的工具有：

```
write
edit
apply_patch
delete
```

典型例子是 aggressive 模式下写入普通项目文件：

```
模型调用 write("src/app.py")
        ↓
aggressive 权限策略判断：
项目内普通写入，可以 ALLOW
        ↓
但是 Session 开启了写前审查
        ↓
系统生成文件 diff
        ↓
用户看到：
Deny
Apply reviewed change
```

此时用户确认的不是“是否授予写文件权限”，因为权限系统已经允许了；用户确认的是：

> 我已经看过这份具体修改，是否把它真正应用到文件中？

`Apply reviewed change` 本质上仍然是一次性的允许，不会生成长期 grant。

## 已有长期授权时也可能出现

例如用户之前已经为 `src/app.py` 保存了 `Allow always`：

```
write("src/app.py")
→ 命中 allow grant
→ 权限结果为 ALLOW
→ 但是仍需检查具体 diff
→ 显示 Apply reviewed change
```

这说明：

```
Allow always
→ 长期允许对这个路径发起写操作

Apply reviewed change
→ 确认本次具体修改内容
```

两者属于不同层次。

## 什么情况下不会出现

如果权限策略本身返回 `ASK`，显示的是普通权限选项：

```
Deny
Allow once
Allow always（工具支持时）
```

即使界面同时展示了 diff，按钮通常仍然叫 `Allow once`，而不是 `Apply reviewed change`。

例如 Standard 模式第一次调用 `write`：

```
权限策略返回 ASK
→ 显示 Deny / Allow once / Allow always
```

Bypass 模式下虽然系统仍可生成写前预览，但不会暂停等待确认，因此不会显示 `Apply reviewed change`。

后台隔离 coder 会关闭逐次写前确认，也不会出现这个选项。

需要特别说明的是，`apply_patch` 虽然支持生成写前预览，但它设置了：

```
allow_auto=False
allow_always=False
```

所以在 Standard 和 Aggressive 下，它的权限策略通常直接返回 `ASK`，显示的是：

```
Deny
Allow once
```

而不是 `Apply reviewed change`。

一句话总结：

> `Apply reviewed change` 出现在“权限已经允许，但具体文件改动仍需人工看 diff 确认”的场景；它是写前审查按钮，不是普通权限授权按钮。
# 那些工具可以看到Reject with feedback选择

FirstCoder 中，只有能够生成写前预览的文件修改工具，才会在权限界面看到 `Reject with feedback` 的提示。目前包括 `write`、`edit`、`delete` 和 `apply_patch`。这些工具执行前，系统可以计算出文件修改后的结果并生成 diff，因此用户不但可以拒绝，还可以附带原因，例如输入 `reject: 请保留原来的异常处理`。系统会取消本次工具调用，并把这段反馈放入失败的工具结果中返回给模型，让模型根据反馈重新生成修改方案。

对应关系是：

| 工具                   | 是否支持写前预览 | 是否显示 `Reject with feedback` |
| -------------------- | -------- | --------------------------- |
| `write`              | 是        | 是                           |
| `edit`               | 是        | 是                           |
| `delete`             | 是        | 是                           |
| `apply_patch`        | 是        | 是                           |
| `shell`              | 否        | 否                           |
| `python_exec`        | 否        | 否                           |
| `fetch`、`web_search` | 否        | 否                           |
| Git 只读工具             | 否        | 否                           |
| MCP 工具               | 否        | 否                           |
| 读取类工具                | 否        | 否                           |

需要严谨说明的是，`Reject with feedback` 当前不是普通权限请求中的正式按钮选项。结构化选项仍然是 `Deny`、`Allow once`，以及可能存在的 `Allow always`。当审批请求携带 `prewrite_review` 时，TUI 会额外显示：

```
Or reply: reject: <feedback>
```

因此它只有在以下条件下才会显示：

```
工具是 write、edit、delete 或 apply_patch
        ↓
Session 开启写前审查
        ↓
成功生成修改预览
        ↓
本次操作需要用户确认
        ↓
显示 reject: <feedback>
```

如果写前预览生成失败，系统会直接阻止操作，不会进入正常审批；如果是 bypass 模式，系统虽然可能生成预览事件，但不会暂停等待用户输入，因此也不会显示这个交互提示。普通命令行适配器能够解析 `reject: ...`，但当前主要是 TUI 会明确把它展示给用户。

一句话概括：

> `Reject with feedback` 是写前审查提供的反馈式拒绝能力，目前只适用于能够生成文件变更预览的 `write`、`edit`、`delete` 和 `apply_patch`。

可能的追问点：

1. `Reject with feedback` 和普通 `Deny` 有什么区别？
2. 用户反馈最终以什么形式返回模型？
3. 为什么 Shell 工具不能使用 `Reject with feedback`？
4. 写前预览是如何生成的？
5. 用户确认前文件内容发生变化怎么办？
6. 为什么 bypass 模式仍然生成写前预览？
7. 写前预览生成失败时系统如何处理？