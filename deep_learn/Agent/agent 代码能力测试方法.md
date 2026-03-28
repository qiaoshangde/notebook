

这两个测试都有完整、开源、可直接运行的**测试工具包 (Evaluation Harness)**。你不需要从零实现——它们已经打包好了，你只需要准备模型的预测结果，然后跑官方脚本就行。

下面我分别详细介绍每个测试的**地点**（在哪里跑）和**方法**（具体怎么跑）。

---

## 一、SWE-bench

### 📍 测试地点

**环境**：Docker 容器 + 你的本地机器（或云服务器）

**官方资源**：
- GitHub：`princeton-nlp/SWE-bench`
- PyPI：`pip install swebench`
- 数据集：[Hugging Face - princeton-nlp/SWE-bench](https://huggingface.co/datasets/princeton-nlp/SWE-bench)

**硬件要求**（官方建议）：
- x86_64 架构（ARM Mac 实验性支持）
- 至少 **120GB 可用磁盘空间**
- 16GB+ RAM
- 8 CPU 核心

### 🛠️ 测试方法（如何跑）

#### 第一步：安装

```bash
git clone https://github.com/princeton-nlp/SWE-bench.git
cd SWE-bench
pip install -e .
```

#### 第二步：准备预测结果文件

你需要把你的模型输出格式化为 JSONL，每一行对应一个任务：

```json
{
  "instance_id": "sympy__sympy-20590",
  "model_name_or_path": "你的模型名",
  "model_patch": "diff --git a/file.py b/file.py\n..."
}
```

- `instance_id`：任务ID，从数据集中获取
- `model_patch`：你的模型生成的 git patch 格式的代码修改

#### 第三步：运行评测

```bash
python -m swebench.harness.run_evaluation \
    --dataset_name princeton-nlp/SWE-bench_Lite \
    --predictions_path /path/to/your/predictions.jsonl \
    --max_workers 8 \
    --run_id my_evaluation
```

**参数说明**：
- `--dataset_name`：可选 `SWE-bench`（完整版）或 `SWE-bench_Lite`（轻量版，约300个任务）
- `--predictions_path`：你的预测文件路径
- `--max_workers`：并行数，建议不超过 CPU 核数的 75%
- `--instance_ids`：可以只测特定任务

#### 第四步：查看结果

结果输出在 `evaluation_results/` 目录下：
- `results.json`：整体指标
- `instance_results.jsonl`：每个任务的详细结果
- 关键指标：`resolved_instances` / `total_instances` = **解决率**

### 💡 核心原理

SWE-bench 的测试逻辑是：
1. 将模型生成的 patch 应用到对应的代码仓库
2. 在 Docker 容器中运行该仓库的测试套件
3. 检查：原来失败的测试（fail-to-pass）是否通过了？原来通过的测试（pass-to-pass）是否仍然通过？
4. 两者都满足 → 任务解决 ✅

---

## 二、Terminal-Bench 2.0 (TB2)

### 📍 测试地点

**环境**：Docker / Daytona / Modal / E2B 等隔离容器

**官方资源**：
- 官网：[tbench.ai](https://www.tbench.ai/)
- Harbor 框架（评测工具）：`pip install harbor==0.1.28`
- EvalScope 集成：`evalscope` 支持 TB2

### 🛠️ 测试方法（如何跑）

#### 方法一：使用 EvalScope（推荐快速上手）

```bash
# 安装
pip install evalscope

# 运行评测
evalscope eval \
    --model YOUR_MODEL \
    --api-url https://your-api-endpoint \
    --api-key YOUR_API_KEY \
    --datasets terminal_bench_v2 \
    --limit 10   # 正式评测去掉这行
```

来源：[Terminal-Bench-2.0 | EvalScope](https://evalscope.readthedocs.io/zh-cn/latest/benchmarks/terminal_bench_v2.html)

#### 方法二：使用 Harbor 框架（更灵活，适合多任务并行）

```bash
# 安装 harbor
pip install harbor==0.1.28

# 运行评测（需要配置你的 agent wrapper）
harbor run \
    --agent-import-path your_agent:AgentClass \
    --dataset terminal-bench@2.0 \
    -n 10 \
    --env docker
```

来源：LangChain 的 Deep Agents CLI 评测实践

#### 方法三：从源码运行

```python
from evalscope import run_task
from evalscope.config import TaskConfig

task_cfg = TaskConfig(
    model='your-model-name',
    api_url='https://your-api-url',
    api_key='your-api-key',
    datasets=['terminal_bench_v2'],
    dataset_args={
        'terminal_bench_v2': {}
    },
)

run_task(task_cfg=task_cfg)
```

来源：[EvalScope 文档](https://evalscope.readthedocs.io/zh-cn/latest/benchmarks/terminal_bench_v2.html)

### 💡 核心原理

TB2 的测试逻辑是：
1. 每个任务是一个**独立的 Docker 容器**，里面配置好特定的文件、依赖、系统状态
2. 给 Agent 一个自然语言指令（例如："编译 SQLite 并生成覆盖率报告"）
3. Agent 在容器内自由操作终端（`ls`、`cd`、`make`、`python` 等）
4. 评测器运行 **pytest 验证脚本**，检查容器的最终状态是否符合任务要求
5. 验证通过 → 任务完成 ✅

### 📊 TB2 任务示例

| 任务类型 | 示例 | 难度 |
|---------|------|------|
| Software Engineering | 编译遗留代码、调试 | 高 |
| Security | 逆向工程二进制文件 | 高 |
| System Administration | Linux 模块配置、文件恢复 | 中高 |
| Git 操作 | 多分支合并冲突处理 | 中 |
| Scientific Computing | DNA 引物生成、模拟优化 | 高 |

来源：[Terminal-Bench 论文摘要](https://arxiv.org/abs/2601.11868) 和 [Terminal-Bench 主题页](https://www.emergentmind.com/topics/terminal-bench-2-0)

---

## 三、快速上手指南

如果你想**自己写一个类似的东西**并测试，这是最简路径：

### 对于 SWE-bench 类（代码修复）
1. 让你的模型接收（issue描述 + 代码仓库）→ 输出 patch
2. 将 patch 按 JSONL 格式保存
3. 跑官方评测脚本，获得解决率

### 对于 TB2 类（终端操作）
1. 让你的 Agent 能够与终端交互（执行命令、读输出）
2. 通过 Harbor 或 EvalScope 对接 TB2 数据集
3. 跑评测，获得 pass rate

### 核心区别总结

| 维度 | SWE-bench | Terminal-Bench 2.0 |
|------|-----------|-------------------|
| 测试对象 | 代码 patch | 终端命令序列 |
| 输入 | issue 描述 + 代码库 | 自然语言任务指令 |
| 输出 | git patch | 一系列终端操作 |
| 验证方式 | 运行单元测试 | 运行 pytest 检查最终状态 |
| 官方工具 | swebench 包 | Harbor / EvalScope |
| 硬件需求 | 高（~120GB） | 中等（取决于并行数） |

---

## 四、高级：在 CI/CD 中自动化评测

JetBrains 的团队分享过如何用 TeamCity 构建 SWE-bench 自动化评测流水线：

- 缓存 Hugging Face 数据集和 Docker 镜像，避免重复下载和限流
- 控制并行度，避免 API 限流
- 自动标记构建结果（✅ Good solution / ❌ Bad solution）

如果你需要频繁测试模型迭代版本，这套思路值得参考。

---

## 总结

**两个测试都有现成的、开箱即用的评测工具**：
- **SWE-bench**：用 `swebench` 包 + Docker，跑的是代码仓库的单元测试
- **TB2**：用 `Harbor` 或 `evalscope`，跑的是容器环境中的 pytest 验证脚本

你想要测试自己写的类似系统，最快的方式就是直接用官方的评测脚本，对接你的模型输出格式即可。官方代码仓库里都有详细示例，跟着跑一遍就能上手。