

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