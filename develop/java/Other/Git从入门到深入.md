1. 基本概念

   - 版本控制系统
   - 中心式版本控制
   - 分布式版本控制系统git，广义上也可以被看作一种 中心式系统
   - 分布式系统也适合 需要官方中心服务器的项目
   - 合并冲突
   - 一些常见的版本控制系统
     - Mercurial  轻量级分布式版本控制系统，python实现，扩展性强，开源
     - Bazaar  分布式的版本控制系统，易用，稳定，灵活，支持插件
     - CVS  Concurrent Versions System 是一种GNU软件包，主要用于在多人开发环境下源码的维护
     - Subversion  集中式代码管理，有一台核心服务器，所有的版本信息都放在服务器上。如果脱离了服务器，开发者基本上可以说是无法工作的

2. 基本命令动作

   - 保存状态

     ```shell
     # 初始化本地仓，提交 
     $ git init
     $ git add .
     $ git commit -m "My first backup"
     
     # 还原
     $ git reset --hard
     
     # 再次保存状态
     $ git commit -a -m "Another backup"
     ```

   - 添加、删除、重命名

     ```shell
     # 添加新文件
     # git add [file_1] [file_2] [file_3]
     $ git add readme.txt Documentation
     
     # 让git忘记某些文件
     # 这些文件如果还没被从系统中删除，Git将会删除它们
     $ git rm kludge.h obsolete.c
     $ git rm -r incriminating/evidence/
     
     # 重命名文件同删除旧文件，并同时添加新文件一样。也有一个快捷方式 git mv
     $ git mv bug.c feature.c
     ```

   - 相对某个时间点 回滚

     ```shell
     # 显示最近提交列表，以及查看他们的SHA1哈希值
     $ git log
     commit 766f9881690d240ba334153047649b8b8f11c664
     Author: Bob <bob@example.com>
     Date:   Tue Mar 14 01:59:26 2000 -0800
     
         Replace printf() with write().
     
     commit 82f5ea346a2e651544956a8653c0f58dc151275c
     Author: Alice <alice@example.com>
     Date:   Thu Jan 1 00:00:00 1970 +0000
     
         Initial commit.
     # ----------------------
     # 哈希值的前几个字符足够确定标识一个提交，完整粘贴也可以
     # 恢复至一个指定的提交状态，并且从记录中永久抹掉所有比 766f新的 提交
     $ git reset --hard 766f
     
     # 只是向简单地回朔到某一个旧状态 --  该操作将把你带回过去，同时也保留较新提交
     # 相当于创建了一条新的分支 branch 
     $ git checkout 82f5
     # 回来当下
     $ git checkout master 
     
     # 注意每次 checkout之前 记得commit 或者 reset
     
     # 支持只恢复特定文件和目录
     # 这种形式的checkout 会不声不响地覆盖当前文件
     $ git checkout 82f5 some.file another.file
     
     # 建议执行命令之前 来一条
     git commit -a
     
     # 跳到以特定字符串开头的提交
     $ git checkout :/"My first b"
     
     # 你也可以回到倒数第五个保存状态：
     $ git checkout master~5 (0-x) 0 - 当前状态
     
     # 撤销上一次commit
     $ git commit -a
     $ git revert 1b6d
     
     # 生成一个log文件
     $ git log > ChangeLog
     ```

   - 查看变更

     ```shell
     # 从上次提交之后你已经做了什么改变
     $ git diff
     
     # 或者自昨天的改变：git 
     $ git diff "@{yesterday}"
     
     # 一个特定版本与倒数第二个变更之间：
     $ git diff 1b6d "master~2"
     
     # 输出结果都是补丁格式，可以用 git apply 来把补丁打上。也可以试一下：
     $ git whatchanged --since="2 weeks ago"
     ```

3. 克隆代码库

   - 两台计算机之间的同步

     ```shell
     # 在机器A上 初始化一个Git仓库并提交你的文件
     # 然后转到机器B上 执行：
     $ git clone A_computer:/path/to/files
     
     # 以创建这些文件和Git仓库的第二个拷贝
     $ git commit -a
     $ git pull other.computer:/path/to/files HEAD
     
     ```

   - 游击版本控制

     ```shell
     你正做一个使用其他版本控制系统(SVN)的项目， 而你非常思念Git？ 
     方法相当于SF.com这边开发创建自己的fork库
     
     #在工作目录(SVN控制下) 初始化一个Git仓库：
     $ git init
     $ git add .
     $ git commit -m "Initial commit"
     
     # 然后克隆它(git控制)：
     $ git clone . /some/new/directory
     
     # 并在这个fork目录(git控制)工作，按你所想在使用Git。
     # 一旦需要和其他每个人同步
     # 转到SVN的工作目录，用其他的版本控制工具同步，并commit到git仓库
     $ git add .
     $ git commit -m "Sync with everyone else"
     
     # 然后转到git控制的目录运行：
     $ git commit -a -m "Description of my changes"
     $ git pull
     
     # 再回到工作目录(SVN控制下)，使用SVN的上载命令，提交代码到中央仓库
     
     # 该命令为Subversion仓库自动化了上面的操作，并且也可以用作导出Git项目到Subversion仓库的替代
     git svn
     ```

4. 使用分支branch

   - 适用场景：外部因素要求必须切换场景，如新特性，紧急版本，bug修复

     ``` shell
     # 在某个目录初始化仓库：
     $ echo "I'm smarter than my boss" > myfile.txt
     $ git init
     $ git add .
     $ git commit -m "Initial commit"
     
     # checkout 一个新branch boss并且修改文件commit
     $ git checkout -b boss  # 之后似乎没啥变化
     $ echo "My boss is smarter than me" > myfile.txt
     $ git commit -a -m "Another commit"
     
     # 切回到原来的 master分支
     $ git checkout master  # 切到文件的原先版本
     
     # 切回到 boss分支
     $ git checkout boss  # 切到适合老板看的版本
     
     # ---------------------------
     # 一个bug 在提交 `1b6d…`
     $ git commit -a
     $ git checkout -b fixes 1b6d
     
     # 修复bug
     $ git commit -a -m "Bug fixed"
     $ git checkout master
     
     # 并可以继续你原来的任务。你甚至可以“合并”到最新修订
     $ git merge fixes
     ```

