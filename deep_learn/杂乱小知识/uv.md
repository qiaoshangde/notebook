仅包括uv包管理的基本用法

```bash
创建虚拟环境，在本项目的根目录创建，虚拟环境随项目走。
uv venv --python 3.11


创建在当前位置的指定文件夹
uv venv myenv
# 或
uv venv ./myenv

```


```bash
激活虚拟环境
.venv\Scripts\activate
```


创建多个隔离环境

```bash
# 1. 创建项目A的开发环境
cd project-a
uv venv .venv --python 3.10
uv pip install flask requests

# 2. 创建项目B的开发环境
cd ../project-b
uv venv .venv --python 3.11
uv pip install django pandas

# 3. 创建临时测试环境
uv venv ~/tmp/test-env --python 3.9
source ~/tmp/test-env/bin/activate
```


```bash
环境同步。
uv sync
```
使用uv管理项目包的话，会有一个项目依赖文件（pyproject.toml）。一旦执行uv sync，就是根据这个项目依赖文件同步虚拟环境。

1. 读取 `pyproject.toml` 或 `requirements.txt` 中的依赖
    
2. 解析依赖关系，生成/更新 `uv.lock` 文件
    
3. **安装/更新/删除**虚拟环境中的包，使其与锁文件完全一致、


### 克隆现有项目时

```bash
# 克隆项目后
git clone https://github.com/example/project.git
cd project

# 只需要一步：创建并同步环境
uv sync
# ✅ 自动创建 .venv
# ✅ 安装所有依赖（与锁文件完全一致）
# ✅ 保证与开发者环境一致
```



1. **`uv add browser-use`** 📦：添加依赖并安装（自动生成锁文件），通常包含命令uv sync




```bash
uv python install 3.11
```
下载制定版本的python


**`uv remove <package>`**：卸载指定的包，并同步从项目配置文件中擦除对应记录。

**`uv pip install <package>`**：兼容传统 `pip` 的底层安装指令，仅将包安装到当前环境中，但**不**修改 `pyproject.toml`。通常用于临时测试。


**`uv run <script/command>`**：智能识别当前项目的虚拟环境并在其中执行命令或脚本（例如 `uv run main.py`），彻底免除手动执行 `activate` 激活环境的繁琐步骤。



`deactivate` 是用于**退出当前激活的 Python 虚拟环境**的命令。



```bash

# 1. 初始化项目
$ uv init weba pp
$ cd webapp

# 2. 添加生产依赖（会写入 pyproject.toml）
$ uv add fastapi uvicorn
✅ 更新 pyproject.toml
✅ 生成 uv.lock
✅ 安装到 .venv

# 3. 添加开发依赖（也会写入，但标记为 dev）
$ uv add --dev pytest black ruff
✅ 更新 pyproject.toml（添加 [tool.uv.dev-dependencies]）

# 4. 临时测试一个包（不记录）
$ uv pip install ipython  # 临时用一下，不污染项目配置

# 5. 查看最终的 pyproject.toml
$ cat pyproject.toml
[project]
name = "webapp"
version = "0.1.0"
dependencies = [
    "fastapi>=0.104.0",
    "uvicorn>=0.24.0",
]

[tool.uv]
dev-dependencies = [
    "pytest>=7.4.0",
    "black>=23.0.0",
    "ruff>=0.1.0",
]
```