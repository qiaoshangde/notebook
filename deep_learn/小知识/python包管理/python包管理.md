
conda 
# conda环境搭建

一、下载Anaconda

下载链接

[Unleash AI Innovation and Value | Anaconda](https://www.anaconda.com/)

在Anaconda Prompt中输入

1、输入 `conda env list` 查看现有环境（`conda info -e`）（`conda info --envs`）也可以查看已经创建的环境

2、创建环境  conda create --name 环境名 python==python的版本

    比如：`conda create --name yolo python==3.9`

    创建了名字为yolo的环境，python版本为3.9     过程中需要输入y确认

3.、输入 `activate name`（name是你想切换的环境）   激活环境 `conda activate name` 也可以

4、删除环境 conda remove -n  your_env_name (虚拟环境名称) --all

5、pip list 可以查看此环境中已经安装的包。

6、conda create --name new_env_name --clone old_env_name‘
复制old_env_name为new_env_name

7、conda clean -i   清除conda缓存

8、在命令行中输入nvidia-smi  可以看到Cuda版本等信息

9、重置base环境：

```
# 看到不同版本的历史
# conda list --revisions

# 0代表要恢复到的版本
conda install --rev 0
# 或者：conda install --revision 0
```

二、pytorch安装

进入pytorch官网    链接：[PyTorch](https://pytorch.org/)

按照你的版本来安装pytorch       （CPU版本和CUDA的版本来安装）   将获取的链接复制到你的环境中运行

验证pytorch是否安装成功    在此环境中进入python   然后输出

`

```
import torch`
torch.__version__` 

torch.cuda.is_available()显示true导入成功
```

如果显示pytorch的版本号，则显示安装成功


**装pytorch12.1**
```
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```


```
conda install pytorch pytorch-cuda=12.1 -c pytorch -c nvidia
```


# uv

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