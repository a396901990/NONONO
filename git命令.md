Git命令
===========

git配置（config）：
-----------------
    
    # 查看版本：  
    git version    

    # 查看当前配置：  
    git config -l

    # 设置用户名，邮箱：  
    git config --global user.name "Dean"  
    git config --global user.email g.xiangyu1990@gmail.com  

    # 解除用户名，邮箱：  
    git config --unset --global user.name "Dean"
    git config --unset --global user.email g.xiangyu1990@gmail.com

    # 设置git命令的别名
    git config --global alias.ci commit
    git config --global alias.co checkout

git仓库（repository）：
-------------
          
    # 创建一个本地的git仓库并命名：  
    git init demo

    # 克隆一个远程的git仓库到指定路径：  
    git clone https://github.com/a396901990/android.git /path/workpsace

git分支（branch）:
-----------------
    
    git branch                      # 查看分支
    git remote show origin          # 查看所有分支
    git branch <branchname>         # 创建新分支
    git checkout <branchname>       # 切换到分支
    git checkout -b <new_branch>    # 创建并切换到新分支
    git branch -d <branchname>      # 删除分支（-D强删） 
    git branch -m <old> <new>       # 本地分支重命名

git差异（diff）：
--------------------
    
    git diff                     # 查看工作目录（working tree）暂存区（index）的差别
    git diff --cached            # 查看暂存起来的文件（stage）与并未提交（commit）的差别
    git diff --staged            # 同上
    git diff HEAD                # 查看最后一次提交之后的的差别（HEAD代表最近一次commit的信息）
    git diff --stat              # 查看显示简略结果(文件列表)
    git diff commit1 commit2     # 对比两次提交的内容（也可以是branch，哈希值）

git查看历史（log）：
----------------------
    
    git log
    git log -3           # 查看前3次修改
    git log --oneline    # 一行显示一条log
    git log -p           # 查看详细修改内容  
    git log --stat       # 查看提交统计信息
    git log --graph      # 显示何时出现了分支和合并等信息

git查看状态（status）：
----------------------------
    
    git status           # 查看你的代码在缓存与当前工作目录的状态
    git status -s        # 将结果以简短的形式输出

git存放（stash）: 
--------------------------
    

    git stash                   #保存当前的工作进度
    git stash save "message"    #保存进度加说明
    git stash list              #显示进度列表
    git stash pop               #恢复最新保存的工作进度，并将恢复的工作进度从存储的列表中删除
    git stash apply             #恢复最新保存工作进度，但不删除
    git stash drop              #删除一个进度，默认删除最新的
    git stash clear             #删除所有
