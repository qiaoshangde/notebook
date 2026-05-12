


# git 用到命令
**常用**
git status
git add .

git commit -m "第一次修改"

git pull 下载并合并  ==  git fetch   加 git merge
 
git push 


```
# 从上游仓库拉取更新
git pull upstream main
# 或分两步
git fetch upstream
git merge upstream/main

# 推送到你的仓库
git push origin main

# 如果需要同时设置，可以使用
git push -u origin main  # 设置 origin 为默认推送
```






```
# 1. 克隆你的仓库
git clone <你的仓库URL>
cd <项目目录>

# 2. 添加上游仓库
git remote add upstream <上游仓库URL>

# 3. 验证
git remote -v
# origin    https://github.com/你的用户名/项目.git (fetch)
# origin    https://github.com/你的用户名/项目.git (push)
# upstream  https://github.com/原作者/项目.git (fetch)
# upstream  https://github.com/原作者/项目.git (push)

# 4. 设置分支跟踪
git fetch upstream
git checkout main
git branch -u upstream/main  # 设置拉取跟踪

# 5. 测试
git pull  # 会从 upstream/main 拉取
git push  # 需要指定 origin，或配置别名
```


```


# 完整写法
git branch --set-upstream-to <远程仓库>/<分支名>
```


**设置当前分支跟踪指定的远程分支**
git branch -u <远程仓库>/<分支名>

**不常用**
git branch -a   查看所有分支情况

git branch -vv   查看分支跟踪情况

git checkout -b my-current-work   创建并切换到新分支   git switch -c my-current-work    （-c代表create）

git checkout master   切换到master分支

git remote add  ___

git remote set-url ___

git diff 对比差异

# 我在GitHub上克隆了一个仓库。现在我要对项目进行修改，并将其上传到我的github上。我修改过程中会修改代码（分阶段）所以我也需要git来帮我进行版本管理，所以请你给我一系列命令。

可以，下面我直接给你一套**最实用的命令流程**。  
你的场景是：

- 你已经从别人的 GitHub 仓库 `clone` 下来了
    
- 现在你要在本地修改
    
- 修改过程是**分阶段进行**
    
- 你想把代码上传到**你自己的 GitHub 仓库**
    
- 同时用 Git 做版本管理
    

我建议你采用这种标准做法：

- 原作者仓库保留为 `upstream`
    
- 你自己的仓库设为 `origin`
    
- 以后你每改一阶段就提交一次 `commit`
    

---

# 一、先进入项目目录

```bash
cd 你的项目文件夹
```

先看看当前远程仓库是什么：

```bash
git remote -v
```

---

# 二、把远程仓库改成标准结构

假设你现在 clone 下来的仓库是原作者的仓库。  
先把它改名为 `upstream`：

```bash
git remote rename origin upstream
```

再检查一下：

```bash
git remote -v
```

这时候应该看到类似：

```bash
upstream  https://github.com/别人/原仓库.git (fetch)
upstream  https://github.com/别人/原仓库.git (push)
```

---

# 三、在 GitHub 上新建你自己的仓库

在 GitHub 网页上新建一个空仓库，比如：

```bash
https://github.com/你的用户名/你的仓库名.git
```

**注意：不要勾选初始化 README、.gitignore、license**，因为你本地已经有项目了。

然后把你自己的仓库加为 `origin`：

```bash
git remote add origin https://github.com/你的用户名/你的仓库名.git
```

再检查：

```bash
git remote -v
```

这时候应该是：

```bash
origin    https://github.com/你的用户名/你的仓库名.git (fetch)
origin    https://github.com/你的用户名/你的仓库名.git (push)
upstream  https://github.com/别人/原仓库.git (fetch)
upstream  https://github.com/别人/原仓库.git (push)
```

---

# 四、看一下当前分支名

```bash
git branch
```

如果当前分支是 `main`，后面就用 `main`。  
如果是 `master`，就把下面命令里的 `main` 换成 `master`。

---

# 五、第一次推送到你自己的 GitHub

```bash
git push -u origin main
```

如果你当前分支不是 `main`，比如是 `master`：

```bash
git push -u origin master
```

`-u` 的作用是把本地分支和远程分支关联起来，以后直接 `git push` 就行。

---

# 六、后续开发建议：新建你自己的开发分支

不要总在 `main` 上直接改，最好新建一个开发分支：

```bash
git checkout -b dev
```

然后推到远程：

```bash
git push -u origin dev
```

以后你的开发就主要在 `dev` 上进行。

---

# 七、每一个阶段修改时的标准 Git 命令

## 1）查看改了什么

```bash
git status
```

如果想看具体代码差异：

```bash
git diff
```

---

## 2）添加修改到暂存区

全部加入：

```bash
git add .
```

如果你只想加某个文件：

```bash
git add 文件名
```

---

## 3）提交一个阶段版本

```bash
git commit -m "第一阶段：完成xxx功能"
```

例如：

```bash
git commit -m "第一阶段：完成登录模块重构"
git commit -m "第二阶段：修复接口调用逻辑"
git commit -m "第三阶段：优化前端页面展示"
```

---

## 4）推送到 GitHub

```bash
git push
```

---

# 八、推荐你每阶段都这样做

你每做完一个小阶段，就执行：

```bash
git status
git add .
git commit -m "阶段说明"
git push
```

这是最稳的版本管理方式。

---

# 九、如果你想回到之前某个版本

先看提交历史：

```bash
git log --oneline
```

会看到类似：

```bash
a1b2c3d 第三阶段：优化前端页面展示
d4e5f6g 第二阶段：修复接口调用逻辑
h7i8j9k 第一阶段：完成登录模块重构
```

## 临时查看旧版本

```bash
git checkout 提交ID
```

比如：

```bash
git checkout d4e5f6g
```

看完后切回来：

```bash
git checkout dev
```

---

# 十、如果你想撤销“还没 commit”的修改

### 撤销某个文件修改

```bash
git checkout -- 文件名
```

### 撤销全部未提交修改

```bash
git checkout -- .
```

如果是新版 git，也可以用：

```bash
git restore .
```

---

# 十一、如果你已经 add 了，但还没 commit，想取消暂存

```bash
git reset
```

取消某个文件暂存：

```bash
git reset 文件名
```

---

# 十二、如果你想回滚到上一次提交

保留代码，只撤销 commit：

```bash
git reset --soft HEAD~1
```

连暂存也一起撤销，但代码还在：

```bash
git reset --mixed HEAD~1
```

彻底回到上一个版本，代码也不要了：

```bash
git reset --hard HEAD~1
```

这个要谨慎。

---

# 十三、如果原仓库更新了，你想同步原作者最新代码

先拉原仓库信息：

```bash
git fetch upstream
```

看原仓库分支：

```bash
git branch -r
```

把原仓库 `main` 合并到你当前分支：

```bash
git merge upstream/main
```

如果原仓库是 `master`：

```bash
git merge upstream/master
```

---

# 十四、最推荐你的实际工作流

## 第一次配置

```bash
cd 你的项目文件夹
git remote -v
git remote rename origin upstream
git remote add origin https://github.com/你的用户名/你的仓库名.git
git remote -v
git push -u origin main
git checkout -b dev
git push -u origin dev
```

---

## 之后每一阶段开发

```bash
git checkout dev
git status
git add .
git commit -m "第N阶段：你这次做的事情"
git push
```

---

## 同步原作者更新时

```bash
git checkout dev
git fetch upstream
git merge upstream/main
git push
```

---

# 十五、如果你嫌上面太多，记住最核心的 6 条就够了

```bash
git status
git add .
git commit -m "这一次修改说明"
git push
git log --oneline
git diff
```

---

# 十六、给你一套可直接替换的模板

把下面这几处换成你自己的：

- `你的用户名`
    
- `你的仓库名`
    
- `main` 或 `master`
    

```bash
cd your-project

git remote -v
git remote rename origin upstream
git remote add origin https://github.com/你的用户名/你的仓库名.git
git remote -v

git push -u origin main

git checkout -b dev
git push -u origin dev

git status
git add .
git commit -m "第一阶段：初始化项目修改"
git push

git status
git add .
git commit -m "第二阶段：完成核心逻辑修改"
git push

git fetch upstream
git merge upstream/main
git push
```

如果你愿意，我下一条可以直接给你一份**“适合直接复制执行的完整命令清单”**，按“第一次配置 / 日常开发 / 回退版本 / 同步原仓库”四部分整理好。


# 将dev合并到main用下面这套就行。

## 一、先切到 `main`

```bash
git checkout main
```

如果你的主分支叫 `master`，就改成：

```bash
git checkout master
```

---

## 二、先把 `main` 更新到最新

```bash
git pull origin main
```

---

## 三、把 `dev` 合并进 `main`

```bash
git merge dev
```

如果没有冲突，就会直接合并成功。

---

## 四、推送到 GitHub

```bash
git push origin main
```

---

# 一套完整命令

```bash
git checkout main
git pull origin main
git merge dev
git push origin main
```

---

# 如果出现冲突

执行 `git merge dev` 后，如果提示冲突：

先看哪些文件冲突了：

```bash
git status
```

打开冲突文件，你会看到类似：

```text
<<<<<<< HEAD
main 分支的内容
=======
dev 分支的内容
>>>>>>> dev
```

你需要手动改成你最终想保留的内容，改完后执行：

```bash
git add .
git commit -m "merge dev into main"
git push origin main
```

---

# 合并完成后，如果你还想继续在 `dev` 开发

通常不用删 `dev`，继续留着就行。  
以后还是在 `dev` 上改，改完再合并到 `main`。

常见流程就是：

```bash
git checkout dev
# 继续开发
git add .
git commit -m "新功能"
git push origin dev

git checkout main
git pull origin main
git merge dev
git push origin main
```

---

# 更稳一点的做法

为了避免本地 `dev` 不是最新，合并前可以先更新一下 `dev`：

```bash
git checkout dev
git pull origin dev
git checkout main
git pull origin main
git merge dev
git push origin main
```

---

# 你这个场景下，最推荐直接用这套

```bash
git checkout dev
git pull origin dev
git checkout main
git pull origin main
git merge dev
git push origin main
```

如果你要，我还可以继续给你一份：

**“dev、main 双分支最标准工作流”**  
包括：开发、测试、合并、打标签、回滚。