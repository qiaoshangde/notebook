


git 用到命令


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