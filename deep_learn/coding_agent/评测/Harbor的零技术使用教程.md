# Harbor 的零技术使用教程

> 面向第一次接触 Agent 评测的读者，以 FirstCoder 和 Aider Polyglot 为例。
>
> 使用顺序：**准备环境 → 准备 Agent 适配器 → 准备题库 → 启动评测命令 → 查看运行结果。**

## Harbor 是什么？

**Harbor 是一个用于评估和优化 Agent、模型的框架，以 Python 包的形式发布，同时提供命令行工具。**

你不需要自己写一个程序，逐个启动题目、运行 Agent、执行测试、保存结果。Harbor 把这些工作统一组织起来。

可以用考试来理解：

| 考试里的东西    | 评测里的东西      | 本项目的例子                |
| --------- | ----------- | --------------------- |
| 考务系统      | Harbor      | 准备环境、安排执行、收集评分        |
| 试卷        | Dataset，任务集 | Aider Polyglot        |
| 一道题       | Task        | 一个具体编程练习              |
| 考生        | Agent       | FirstCoder            |
| 考生使用的推理能力 | 底层模型        | 实际配置的模型服务             |
| 考场        | Environment | Docker 容器             |
| 判卷程序      | Verifier    | 检查代码是否通过测试            |
| 一次答题尝试    | Trial       | FirstCoder 对某一道题的一次尝试 |
| 一场完整考试    | Job         | 一批任务及其执行结果            |

Harbor 不是模型，也不是 Aider Polyglot 的另一个名字。它负责“怎么组织评测”；题库负责“测什么”；FirstCoder 负责“完成任务”。

官方依据：[Harbor 介绍](https://www.harborframework.com/docs)、[核心概念](https://www.harborframework.com/docs/core-concepts)。

## 一、准备环境

### 1.1 需要准备什么？

| 东西              | 作用                | 是否必须          |
| --------------- | ----------------- | ------------- |
| Python 3.12     | 在主机运行 Harbor 和适配器 | 本文固定使用它       |
| Harbor 0.18.0   | 组织评测              | 是             |
| Docker Desktop  | 在本机运行任务容器         | 本文的本地运行方式需要   |
| FirstCoder 源码   | 被评测的 Agent        | 是             |
| Harbor 格式题库     | 提供题目、环境和验证程序      | 是             |
| 模型服务与 API Key   | 让 FirstCoder 调用模型 | 正常模型评测需要      |
| 反馈插件            | Aider 测试失败后追加修复机会 | 要复现项目的反馈模式时需要 |
| 下载缓存、wheelhouse | 减少依赖重复下载          | 可选，先跑通再配置     |

这里有两个 Python 环境：

- **主机环境**：安装 Harbor，以及加载 FirstCoder 适配器所需的依赖。
- **题目容器里的环境**：由适配器安装 FirstCoder，让它在题目目录中运行。

本机装好了 FirstCoder，不代表题目容器里也装好了；容器安装步骤由适配器执行。

### 1.2 安装 Harbor

找一个文件夹作为评测工作目录，使用 **Python 3.12** 环境，直接安装 Harbor：

```powershell
python -m pip install "harbor==0.18.0"
```

安装后即可使用 `harbor` 命令，例如：

```powershell
harbor run --help
```

本文固定 0.18.0 是为了配合 FirstCoder 现有适配器。后续评测 FirstCoder 时，在其项目根目录执行示例命令；`python` 和 `harbor` 应来自同一个 Python 环境。


### 1.3 启动 Docker Desktop

在本机安装并启动 [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/) 。安装完成后打开 Docker Desktop，等待引擎启动，并确认使用 Linux 容器。

在 PowerShell 中检查：

```powershell
docker version
```

如果能看到 Client 和 Server 信息，说明 Docker Desktop 已正常运行。

### 1.4 配置模型服务和密钥

#### 创建本地配置文件

在 FirstCoder 根目录中，用编辑器创建 `.env.harbor`。

填入以下格式：

```dotenv
FIRSTCODER_PROVIDER_NAME=my_provider
FIRSTCODER_MODEL=替换为服务端真实模型ID
FIRSTCODER_BASE_URL=https://替换为真实服务地址/v1
FIRSTCODER_API_KEY=替换为自己的密钥
FIRSTCODER_DISABLE_GLOBAL_SKILLS=1
```

| 字段 | 怎么填写 |
|---|---|
| `FIRSTCODER_PROVIDER_NAME` | FirstCoder 内部使用的 Provider 标识，例如 `my_provider`；不是密钥 |
| `FIRSTCODER_MODEL` | 服务端真实支持的模型 ID，不是自己随便起的名字 |
| `FIRSTCODER_BASE_URL` | 与项目适配器兼容的模型服务地址；这里使用 OpenAI-compatible 配置方式 |
| `FIRSTCODER_API_KEY` | 模型服务的 API 密钥 |
| `FIRSTCODER_DISABLE_GLOBAL_SKILLS` | 设为 `1`，避免本机全局技能影响评测环境 |

这些内容以环境变量的形式传入容器，由 FirstCoder 适配代码读取；变量名由 Agent 或适配器约定，不是 Harbor 的统一配置字段。换成其他 Agent，就按它要求的变量名填写。`.env.harbor` 也只是本项目采用的文件名。

#### 配置如何传入容器

后面的命令同时使用：

- `--env-file`：让 Harbor 加载本地配置。
- `--agent-env`：明确把需要的变量传给 Agent。

## 二、准备 Agent 适配器

### 2.1 是否需要自己编写？

- 使用 Harbor 已支持的 Agent：直接选用现成接入。
- 使用自研 Agent：需要提供适配器，告诉 Harbor 怎样安装和启动它。
- FirstCoder 已经有适配器，不需要重新编写。

FirstCoder 适配器负责在任务容器中安装 FirstCoder，将题目交给非交互的 benchmark 入口运行，并导出执行日志。后面通过 `-a` 指定：

```text
benchmark.harbor.shared.firstcoder_agent:FirstCoderHarborAgent
```

### 2.2 准备依赖并检查适配器

下面前三步是准备操作，后两步是可选检查。按顺序执行：

```powershell
# 1. 准备：进入 FirstCoder 根目录，让后面的“.”指向这个项目。
Set-Location "E:\agent\后端agent开发资料\FirstCoder"

# 2. 准备：以可编辑方式安装当前 FirstCoder 及其依赖，不是运行测试。
python -m pip install -e .

# 3. 准备：把项目根目录加入 Python 模块搜索路径，让 Harbor 能找到 benchmark 里的适配器。
$env:PYTHONPATH = (Get-Location).Path

# 4. 可选检查：尝试导入适配器，检查模块路径、导入所需依赖和接口是否存在。
# 输出 adapter import OK 说明可以导入；不会启动 Agent、Docker 任务或调用模型。
python -c "from benchmark.harbor.shared.firstcoder_agent import FirstCoderHarborAgent; print('adapter import OK')"

# 5. 可选检查：加载 .env.harbor，检查 Docker、模型配置、网络和缓存等运行条件。
# 可能访问模型服务进行连通性和模型目录检查，但不执行正式答题。
python -m benchmark.harbor.shared.preflight --env-file .env.harbor
```

为什么要设置 `PYTHONPATH`？因为适配器在项目的 `benchmark/` 目录下，需要让当前 Python 可以导入这部分代码。

- 导入测试输出 `adapter import OK`。
- 预检的必要项目通过，没有需要解决的 `fail`。

## 三、准备题库

### 3.1 Aider 题目格式不一定适配 Harbor，怎么办？

**Aider 原始题目格式不一定适配 Harbor。直接下载 Harbor 已适配的 Aider Polyglot 数据集，就不用自己改格式。** 查找和下载命令见下面的 3.2、3.3。

如果只有原始题库，需要把每道题整理成 Harbor 任务，补齐以下内容：

| 文件 | 具体怎么改 |
|---|---|
| `instruction.md` | 放入原题需求，说明需要修改的文件和接口要求，不放参考答案 |
| `task.toml` | 配置 Agent、验证程序的超时和任务环境参数，字段按所用 Harbor 版本填写 |
| `environment/Dockerfile` | 安装题目所需语言环境和依赖，把初始代码放进工作目录；也可配置已有镜像 |
| `tests/` | 保留原题测试，并增加 `test.sh`：执行测试，将判分写入 `/logs/verifier/reward.txt`；这类二值任务通过写 `1`，未通过写 `0` |

也就是说，**不需要重新出题，主要是把原题的需求、初始代码、运行环境和测试接到 Harbor 规定的入口上。** 安装失败或环境异常应保留错误信息，不要全部伪装成代码测试失败。

适配完先选一道题，用已知正确和错误的实现分别检查是否得到预期评分，再批量处理其他题。原测试和判分标准不要为了适配而随意修改。

本次使用现成适配版即可；以上是自己转换题库时才需要做的工作。格式参考：[Harbor 官方任务说明](https://www.harborframework.com/docs/tasks)。

### 3.2 查看当前版本可用的数据集

```powershell
harbor dataset --help
harbor dataset list
```

在输出中找到 Aider Polyglot，记录它的真实标识和版本。不同 Harbor 版本的注册表命名可能不同，不要把网上的标识直接当作当前一定可用。



### 3.3 两种运行入口，选择一种

**注册题库入口：**用 `-d` 指定真实数据集标识，Harbor 可以按注册信息下载。

```text
-d "从 dataset list 确认的完整标识和版本"
```

**本地题库入口：**已有 Harbor 格式数据集时，用 `-p` 指定它的目录。

```text
-p "已经准备好的本地Harbor题库目录"
```

### 3.4 当前项目已有本地题库

包含 **225 道 Harbor 格式任务**，每道题都有 `task.toml`、`instruction.md`、`tests/test.sh` 和 `environment/Dockerfile`。

在 FirstCoder 根目录运行时，指定：

```text
-p ".local/harbor-datasets/aider-polyglot"
```

## 四、启动评测命令

### 4.1 先跑一道题

在 FirstCoder 根目录运行。命令的基本格式是：

```text
harbor run -d "数据集标识@版本" -i "任务名称" -a "Agent适配器" -m "模型标识" -n 1 -k 1 -o "结果目录"
```

当前项目已经有本地题库，直接用 `-p` 替代 `-d`：

```text
harbor run -p ".local/harbor-datasets/aider-polyglot" -i "任务名称" -a "benchmark.harbor.shared.firstcoder_agent:FirstCoderHarborAgent" -m "模型标识" -n 1 -k 1 -o "benchmark/runs/harbor/aider-smoke"
```

上面展示核心参数。使用 FirstCoder 的模型配置和 Aider 反馈修复时，完整命令如下；将“任务名称”和“模型标识”替换为实际值：

```text
harbor run -p ".local/harbor-datasets/aider-polyglot" -i "任务名称" -a "benchmark.harbor.shared.firstcoder_agent:FirstCoderHarborAgent" --plugin "benchmark.harbor.aider_polyglot.aider_feedback_plugin:AiderFeedbackPlugin" --env-file ".env.harbor" --agent-env 'FIRSTCODER_PROVIDER_NAME=${FIRSTCODER_PROVIDER_NAME}' --agent-env 'FIRSTCODER_MODEL=${FIRSTCODER_MODEL}' --agent-env 'FIRSTCODER_BASE_URL=${FIRSTCODER_BASE_URL}' --agent-env 'FIRSTCODER_API_KEY=${FIRSTCODER_API_KEY}' --agent-env "FIRSTCODER_DISABLE_GLOBAL_SKILLS=1" -m "模型标识" -n 1 -k 1 --ak max_tool_rounds=120 --ak max_turn_seconds=1800 --agent-setup-timeout-multiplier 3 -o "benchmark/runs/harbor/aider-smoke" -y
```

### 4.2 命令参数说明

| 参数                                 | 后面填写什么                         | 作用                       |
| ---------------------------------- | ------------------------------ | ------------------------ |
| `-d`                               | 已注册的数据集标识及版本                   | 选择并下载注册题库                |
| `-p`                               | 本地题库目录                         | 使用本地题库，与 `-d` 二选一        |
| `-i`                               | 实际任务名称                         | 只运行指定题目；跑全套时去掉这一项        |
| `-a`                               | Agent 名称或适配器导入路径               | 指定谁来做题                   |
| `--plugin`                         | 插件导入路径                         | 这里启用 Aider 测试反馈修复        |
| `--env-file`                       | 环境变量文件路径，例如 `.env.harbor`      | 加载模型配置                   |
| `--agent-env`                      | `变量名=值`，每个变量写一项                | 将配置传入 Agent 运行环境         |
| `-m`                               | 与实际配置一致的模型标识                   | 记录使用的模型                  |
| `-n`                               | 并发数，例如 `1`                     | 同时运行多少个任务                |
| `-k`                               | 尝试次数，例如 `1`                    | 每道题安排多少次尝试               |
| `--ak`                             | 适配器参数，例如 `max_tool_rounds=120` | 设置 FirstCoder 工具轮数、时间等限制 |
| `--agent-setup-timeout-multiplier` | 倍数，例如 `3`                      | 调整安装 Agent 阶段的超时         |
| `-o`                               | 结果目录                           | 保存本次运行结果；不同批次使用不同目录      |
| `-y`                               | 不填值                            | 跳过启动前的交互确认               |

### 4.3 Aider 反馈插件做什么？

不启用这个插件时，不能假定 Harbor 会自动把验证错误交给 FirstCoder 再做一遍。

本项目的反馈模式大体是：

```text
首次交给 FirstCoder 的是任务需求
    ↓
FirstCoder 修改代码
    ↓
独立验证程序运行
    ↓
若符合反馈条件：把测试错误交给同一个 Session
    ↓
FirstCoder 修复一次
    ↓
再次验证，得到最终结果
```

当前实现的重要边界：

- 只在满足插件条件时追加一次修复，不是无限重试。
- 明确的 `reward=0` 是正常反馈入口。
- 代码还兼容一种没有 reward 文件、但验证输出明确包含 CMake 编译失败及错误标记的情况；不是所有缺失 reward 都能重试。
- 独立 verifier 环境等不支持原地修复的任务，会保留 Harbor 原本执行方式。
- 首次运行时，适配器不主动读取验证文件并注入提示词；反馈内容在验证后才提供。这个边界不应夸大成容器中任何情况下都无法接触测试。
- 不要给 Terminal-Bench 随意启用这个插件，否则改变了评测协议。

特别区分：`-k 1` 是一次 Trial；一次 Trial 内仍可能有插件触发的反馈修复。

### 4.4 跑完整 225 题

在前面的命令基础上：

1. 删除 `-i "任务名称"`，表示不再只筛选那一道题。
2. 使用新的输出目录，避免混淆运行结果。
3. 核对解析出的数据集版本和任务总数。
4. 初期保留 `-n 1`；确认资源和服务额度后，再考虑改成 2 或 4。
5. 继续明确保留或关闭同一反馈协议，不要不同批次随意改变。
6. 检查电源、休眠、网络和模型额度，避免长时间运行被中断。

不要把“并发 4”理解成一个 Agent 内部四个线程，也不要理解成子 Agent 数量。它指 Harbor 同时安排多少个任务尝试。

如果评测失败，只重跑失败题也要记录范围和次数。不能将反复重跑直到成功的结果，当作没有重试的一次性成绩。

### 4.5 中止与恢复

可以在运行 Harbor 的 PowerShell 中按 `Ctrl+C` 请求中止。

但不要理解为所有容器、模型请求和远程操作都会瞬间取消。停止后查看 Harbor 输出及 Docker Desktop，确认是否还残留本次任务的容器。处理前核对名称和归属，不要批量删除全部 Docker 资源。

“重新运行命令”“Harbor 恢复旧 Job”“FirstCoder 恢复旧 Session”“Aider 插件在同一题内反馈修复”是不同机制。

不要假定重新运行原命令会自动接着旧 Job。恢复前运行当前版本的 `harbor --help`，查看对应恢复命令与条件，保留原配置、结果和错误日志，再决定如何继续。

## 五、查看运行结果

### 5.1 找到结果目录

先查看刚才的输出目录：

```powershell
Get-ChildItem -LiteralPath "benchmark/runs/harbor/aider-smoke"
```

Harbor 可能在指定父目录下面再创建一个具体的运行目录。按终端提示确认本次 Job 的真实目录。

典型结构可以理解为：

```text
本次Job目录/
├── config.json            本次配置
├── result.json            整体结果
└── 各个Trial目录/
    ├── result.json        单次尝试结果
    ├── agent/             Agent 日志和可能的 Session 导出
    └── verifier/          验证程序日志和评分文件
```

实际文件名以当前运行产物为准。这是结构示意，不是要求手动创建这些文件。

### 5.2 判断运行是否成功

1. 环境是否准备成功？
2. FirstCoder 是否安装成功？
3. 模型是否成功响应，Agent 是否正常执行？
4. 验证程序是否完成并产生评分？
5. 最终 reward 是否代表通过？

对于本项目 Aider 这次历史使用的二值评分，`1` 代表通过，`0` 代表明确未通过。其他任务可能定义其他评分方式，不能把所有 Harbor benchmark 都理解成同一种评分。

**程序退出、容器启动成功、Agent 说“完成了”，都不等于测试通过。**

### 5.3 使用项目汇总器

```powershell
python -m benchmark.harbor.shared.summarize "本次Job目录的完整路径"
```

### 5.4 本项目的历史 Aider 成绩

项目保留的历史报告与顶层结果记录为：

| 项目 | 历史结果 |
|---|---:|
| 总任务 | 225 |
| 有明确 reward | 221 |
| 通过 | 213 |
| 明确未通过 | 8 |
| 没有明确判分 | 4 |
| 有效判分通过率 | 213 / 221 = 96.38% |
| 端到端通过率 | 213 / 225 = 94.67% |

报告记录使用 `gpt-5.6-luna`、high 推理强度，并启用了测试反馈修复，运行中也有异常重试和恢复记录。

可以说：“我通过 Harbor，在本地适配的 Aider Polyglot 任务集上评测 FirstCoder，启用了测试反馈后的单次修复，最终 225 道任务中通过 213 道。”

不要说：“首次代码生成通过率 96.38%”“官方排行榜第一”“当前分支刚刚重跑也是这个成绩”。没有对应证据。

还要说明：四道无明确判分任务，不能不看日志就全部归因于网络。它们不是四道明确的代码测试失败，也不能当作通过。

## 附录：可选优化与其他题库

### 下载缓存与 wheelhouse

第一次先不要加入这些优化。正常跑通后再配置。

项目适配器支持共享下载缓存：复用的是下载的依赖包，不是让所有任务共享同一个虚拟环境。每个 Trial 仍使用自己的安装环境。

还支持 wheelhouse，可以理解成“提前准备好的一批依赖安装包”，减少容器临时联网下载的需求。

对应入口：

```powershell
python -m benchmark.harbor.shared.prepare_wheelhouse --help
python -m benchmark.harbor.shared.preflight --help
```

项目预检工具可以生成 `--mounts` 所需参数。只挂载专门的缓存目录；wheelhouse 按项目方案只读挂载，不挂载个人目录或密钥文件。

缓存不能保证所有网络访问都消失：模型 API、题目语言依赖、镜像下载可能仍需要网络。

### Terminal-Bench 固定集示例

这不是 Aider 全量评测的必要步骤，只用于理解 Harbor 可以承载另一套题库。

项目提供固定六题运行器。下面通过 `Get-Command` 获取当前 Harbor 的路径并传给它：

```powershell
python -m benchmark.harbor.terminal_bench.run_terminal_bench_ab `
  --harbor-executable (Get-Command harbor).Source `
  --env-file .env.harbor `
  --task chess-best-move `
  --label first-try `
  --dry-run
```

`--dry-run` 用于查看将执行的命令，不启动正式题目评测。脚本的预检仍可能访问网络或准备缓存，不等于完全没有副作用。

确认命令、模型、任务和费用预算后，才考虑去掉 `--dry-run`。不加 `--task` 会选择固定的六道题。

它固定使用项目约定的数据集版本和单并发配置，并不是“完整 Terminal-Bench 成绩”。也不要给它添加 Aider 反馈插件。

## 参考资料与项目入口

### 官方资料

- [Harbor 首页](https://www.harborframework.com/)：框架定位。
- [Harbor 安装与运行指南](https://www.harborframework.com/docs/getting-started)：通用使用入口；注意与固定版本区分。
- [Harbor 核心概念](https://www.harborframework.com/docs/core-concepts)：Task、Dataset、Agent、Trial、Job。
- [自定义 Agent 接入](https://www.harborframework.com/docs/agents)：不修改框架也能接入自己的 Agent。
- [Harbor 0.18.0](https://pypi.org/project/harbor/0.18.0/)：本文沿用的版本和 Python 要求。
- [Docker Desktop Windows 安装指南](https://docs.docker.com/desktop/setup/install/windows-install/)：系统要求、安装和启动。
- [Aider Polyglot 官方介绍](https://aider.chat/2024/12/21/polyglot.html)：题库来源与评测背景。

### FirstCoder 项目入口

- [Harbor 项目使用说明](/E:/agent/后端agent开发资料/FirstCoder/benchmark/harbor/README.md)：项目约定与运行示例。
- [FirstCoder 适配器](/E:/agent/后端agent开发资料/FirstCoder/benchmark/harbor/shared/firstcoder_agent.py)：容器内安装和启动。
- [预检工具](/E:/agent/后端agent开发资料/FirstCoder/benchmark/harbor/shared/preflight.py)：环境检查。
- [离线汇总器](/E:/agent/后端agent开发资料/FirstCoder/benchmark/harbor/shared/summarize.py)：分类统计结果。
- [Aider 反馈修复实现](/E:/agent/后端agent开发资料/FirstCoder/benchmark/harbor/aider_polyglot/aider_feedback_trial.py)：同一 Trial 内追加修复。
- [Terminal-Bench 固定集运行器](/E:/agent/后端agent开发资料/FirstCoder/benchmark/harbor/terminal_bench/run_terminal_bench_ab.py)：六道回归任务的包装命令。

## 使用顺序总结

1. **准备环境**：Python 3.12、Harbor、Docker Desktop，以及模型服务和密钥。
2. **准备 Agent 适配器**：现成 Agent 直接选用，自研 Agent 需要接入；FirstCoder 已经具备。
3. **准备题库**：使用注册数据集，或本地 Harbor 格式任务集。
4. **启动评测命令**：填写题库、Agent、模型、并发、尝试次数和输出目录等参数。
5. **查看运行结果**：检查评分和日志，区分任务未通过与环境、模型服务等异常。
