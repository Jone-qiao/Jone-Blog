极客时间《玩转Git三剑客》学习笔记

一、版本控制系统 VCS（Version Control System）

分布式 VCS -- Git

文档：https://git-scm.com/book/zh/v2

集中式 VCS -- Svn



Git 本地数据管理，大概可以分为三个区

- 工作区（Working Directory）：是可以直接编辑的地方。
- 暂存区（Stage/Index）：数据暂时存放的区域。
- 版本库（commit History）：存放已经提交的数据。



二、使用 Git 之前需要做的最小配置

    git --version                # git 版本查看

    git config --local --list     # 只针对某个仓库有效
    git config --global --list    # 对当前登录用户所有仓库有效
    git config --system --list    # 对系统所有登录的用户有效

    git config --global user.name "Jone"
    git config --global user.email "jone@qiaohk.com"



三、建立 Git 仓库

    git init                 # 以当前文件夹建立仓库
    git init projectName     # 新建一个文件夹作为仓库

    # 配置当前仓库的用户名和邮箱
    git config --local user.name "Jone"
    git config --local user.email "jone@qiaohk.com"
    # 查看 local 配置信息
    git config --local --list

    #【暂存区】添加/删除文件
    git add fileName              # 添加指定的文件到【暂存区】
    git add -u                    # 添加所有 git 管理文件到 【暂存区】
    git add .                     # 同上

    git status                    # 查看【暂存区】状态
    git commit -m'Add New File'   # 提交【暂存区】中未提交的文件到【版本历史库】，并添加提交理由
    git log                       # 查看 commit 提交日志



四、给文件重命名的简便方式

    git mv oldFile newFile   # 修改文件名字



五、通过 git log 查看版本演变历史

    git log -n4 --oneline --all --graph
    git help --web log                # 查看更多

    # 从【工作区】直接提交到【版本历史库】，跳过【暂存区】,前提【版本历史库】中存在相应的文件
    git commit -am'Describe'  



六、gitk：通过图形界面工具来查看版本历史

    brew install git-gui     # MAC 安装
    gitk --all               # 使用命令



七、探秘 .git 文件夹

    HEAD             # 仓库正在工作在哪个分支上
    config           # 配置信息，包含：user.name和 user.email 等
    refs/heads       #【分支】信息
    refs/tags        #【里程碑】信息
    objects          # 对象  tree/commit/blob



八、commit、tree 和 blob 三个对象之间的关系

- commit ：快照，对应一个 tree【快照内容文件夹】；
- tree： 文件夹，里面可以有 tree【文件夹中的文件夹】 和 blob【文件夹中的文件】；
- blob： 文件，最小单位，和文件名无关，只要文件内容一样就是一个 blob



    ☁  .git [master] git cat-file -p bd5aaca52365debdc
    tree 6bea4ae8b1c859589ff60f7f9cb5b5ec6831b17e
    parent fb2181d0144142c8ae7d96aaa023bf886c8eade8
    author Jone <jone@qiaohk.com> 1608628117 +0800
    committer Jone <jone@qiaohk.com> 1608628117 +0800
    
    add  readme
    ☁  .git [master] git cat-file -p 6bea4ae8b1c859589ff60f7f9cb5b5ec6831b17e
    100644 blob 6b3f348f794b8d5970135963554d9e92225afff2	2.jpg
    100644 blob f8ed28af67c30c6f55e02c093f5d93c785bae621	readme
    ☁  .git [master] git cat-file -t bd5aaca52365debdc
    commit
    ☁  .git [master] git cat-file -t 6bea4ae8b1c859589ff60f7f9cb5b5ec6831b17e
    tree
    ☁  .git [master] git cat-file -t 6b3f348f794b8d5970135963554d9e92225afff2
    blob

    git cat-file -p *****          # 对象内容
    git cat-file -t *****          # 对象类型



九、分离头指针

    git checkout 8983e68                 # 基于某个 commit 版本创建分离头指针
    git branch -av                       # 查看全部分支，v：本地分支；a：远程分支
    git checkout master                  # 切换为 master 分支
    git branch branchName 8983e68        # 为分离头指针建立分支
    git branch -D branchName             # 删除分支
    git branch merge branchName          # 合并分支

    # 分支比较
    git diff master branchName    
    git diff master master^
    git diff master master~1   
    
    git diff HEAD HEAD^^
    git diff HEAD HEAD~2 

- HEAD 可以指向一个分支，也可以指向一个 commit ，也就是分离头指针； 
      git branch -b [分支名] [分支名/commitId]     
      # 基于【分支/commit】创建新分支，-b 创建成功后切换到新分支上



十、怎么删除不需要的分支

    git branch -d branchName
    git branch -D branchName

- 用 -d 报 error：The branch is not fully merged，是指这个分支不曾合入到其他任何分支。在日常开发中，我们通常赋予有意义的分支名，Git判断本分支没和任何别的分支合并，意味这删除存在风险【文件丢失】。它也提供我们-D的方式，如果确定无风险就用-D 。



十一、如何修改 Commit 时的 Message

最新 Commit 的 Message

    git commit --amend           # amend 【修正】

老旧 Commit 的 Message

    git rebase -i commitID       # commitID 为【需要修改的 commit 的上一个 commit ID】
    
    r

第一个 Commit 的 Message

    git rebase -i --root
    
    r



十二、合并多个 Commit

连续的 Commit

    git rebase -i <commitID>       # commitID 为【需要修改的 commit 的上一个 commit ID】

    pick 9530306 add newfile            # 1
    s a214a6b edit newfile              # 2
    pick 1adc1ef first edit readme      # 3
    
    # 向上合并



间隔的 Commit

    git rebase -i <commitID>       # commitID 为【需要修改的 commit 的上一个 commit ID】

- 添加 pack ，改变 pack 的顺序，使用 s 合并；

    git rebase --continue

- 修改 message 



十三、 差异比较

    git diff --cached         #【暂存区】和【Head】
    git diff                  #【工作区】和【暂存区】差异比较

    # 两个 commit 比较
    git diff <commitID> <commitID> -- <file>
    git diff <branchName> <branchName> -- <file>



十四、让 【暂存区】 恢复成和 【HEAD】 一样

    git reset HEAD          # 取消【暂存区】全部文件的更改
    git reset HEAD <file>   # 取消【暂存区】部分文件的更改



十五、让 【工作区】 恢复成和 【暂存区】 一样

    git checkout -- <file>



十六、Commit 回退

    git reset --hard <commitID>

- 【工作区】 和 【暂存区】的内容都会回退到该 【commit】 时的内容，后续的修改将被丢弃；



十七、正确删除文件的方法

    git rm fileName               # 删除【工作区】文件，并且将这次删除放入【暂存区】。
    git rm -f fileName            # 删除【工作区】和【暂存区】文件，并且将这次删除放入【暂存区】。



十八、Stash 开发中临时加塞了紧急任务怎么处理？

    git stash              # 使用 stash
    git stash list         # 查看 stash 记录 
    
    git stash apply        # 保留 stash 记录，可以重复使用
    git stash pop          # 删除 stash 记录



十九、指定不需要 git 管理的文件

    .gitignore

- 末尾带 \ 表示文件夹；



二十、如何将 git 备份到本地

    # 哑协议（没有进度条）
    git clone --bare <path> ya.git
    # 智能协议（有进度条）
    git clone --bare file://<path> zhineng.git

- --bare：不带【工作区】

    git remote -v
    git remote add <backupName> file://<backupPath>
    git push --set-upstream <backupName> <branchName>

    git remote show                                      # 显示某个远程仓库的信息：
    git remote rm <backupName>                           # 删除远程仓库
    git remote rename <oldBackupName> <newBackupName>    # 修改仓库名



二十一、GitHub

    git branch -m master main   # 使用 main 替换 master，-M 移动/重命名一个分支，即使目标已存在


