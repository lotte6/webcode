---
title: git
categories: tech
tag: hide
date: 2018-12-18 21:54:48
tags:
---
<font color="red" size='10'>安装</font>
```
git checkout . && git clean -xdf 放弃修改

git 源码安装
yum install curl-devel expat-devel gettext-devel  openssl-devel perl-devel zlib-devel asciidoc xmlto texinfo docbook2X

ln -s /usr/bin/db2x_docbook2texi /usr/bin/docbook2x-texi

tar -zxf git-2.0.0.tar.gz $ cd git-2.0.0
make configure 
./configure --prefix=/usr 
make all doc info 
make install install-doc install-html install-info

git init 
git add *.c
git add README
git commit -m 'initial project version'


git log     命令显示从最近到最远的提交日志
git log --pretty=oneline

2.查看提交历史并列出修改内容 -p, -2表示只显示两次提交记录。
git log -p -2

3.显示被提交的文件名：
git log --stat

4.将每次提交的commitCode和commitComment单行显示：
git log --pretty=oneline

5.显示某次提交的内容：
git show commitCode
git show commitCode --stat
git show commitCode Filename
 　　6.查看某行代码( 如fileName文件中函数xxx_notify() )的提交历史：
　　 git blame fileName | grep xxx_notify
git reset --hard HEAD^ 恢复到上个版本（在Git中，用HEAD表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^）
git reset --hard 3628164 commit id 恢复到最新版，或者某一个版本
git reset origin/master --hard
git checkout -- .
git reset HEAD

git reflog 记录每一次操作


git status 状态
git diff file
it checkout -- readme.txt         工作区的修改全部撤销（命令中的--很重要，没有--，就变成了“创建一个新分支”的命令）
git reset HEAD file                    可以把暂存区的修改撤销掉

git rm --cached readme.txt 删除暂存

删除文件
git rm test.txt
rm 'test.txt'
git commit -m "remove test.txt"
git checkout -- test.txt 误删恢复到最新版

git mv
git commit --amend重新提交

远程########################

[receive]
    denyCurrentBranch = ignore


git clone ssh://git@192.168.0.230:2222/app/sample.git
ssh://git@192.168.169.58/home/gittets


git remote add [shortname] [url] 添加远程仓库
git remote add origin git@github.com:name/learngit.git     （也可以把一个已有的本地仓库与之关联）

git fetch origin 更新
git remote show [remote-name] 查看远程仓库
git push origin master 提交
git push -u origin master 推送到远程仓库 第一次加上-u，关联起来
git pull 抓取远程合并到本地
git last 最后提交信息

git remote     查看远程库的信息
git remote -v     显示详细的信息

git push origin master     推送
git push origin dev          分支推送到远程库对应的远程分支上(先推送master)


多人协作##########################
git checkout -b branch-name origin/branch-name     本地创建和远程分支对应的分支（本地和远程分支的名称最好一致）
git branch --set-upstream dev origin/dev 与他人同时修改文件时，先pull下载最新的分支，然后修改再推送，如果无法pull下来最新的，要执行此命令，指定本地 dev与远程orgin/dev分支的连接。
多人协作的工作模式通常是这样：
首先，可以试图用git push origin branch-name推送自己的修改；
如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；
如果合并有冲突，则解决冲突，并在本地提交；
没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！
如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。
这就是多人协作的工作模式，一旦熟悉了，就非常简单。



git config --global alias.co checkout 别名
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status


.git/config 添加以下内容
[receive]
      denyCurrentBranch = ignore

分支#############################
git checkout -b dev 创建新分支 命令加上-b参数表示创建并切换，相当于以下两条命令

git branch dev
git checkout dev

git branch 查看当前分支(当前分支前面会标一个*号 )

git checkout master 切换回master分支
git merge dev     合并指定分支到当前分支
git branch -d dev 删除dev分支了

#######################################
查看分支：git branch
创建分支：git branch <name>
切换分支：git checkout <name>
创建+切换分支：git checkout -b <name>
合并某分支到当前分支：git merge <name>
删除分支：git branch -d <name>
######################################

冲突
git status 查看冲突文件，并且修改冲突内容后重新提交
git log --graph --pretty=oneline --abbrev-commit     分支的合并情况

git merge --no-ff -m "merge with no-ff" dev     --no-ff参数，表示禁用Fast forward，fast forward合并就看不出来曾经做过合并
git log --graph --pretty=oneline --abbrev-commit 


git stash     可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作
git stash list     查看暂存列表

一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；
另一种方式是用git stash pop，恢复的同时把stash内容也删了：
git stash apply stash@{0}     恢复指定的stash

git branch -D feature-vulcan 强行删除未提交分支

强制覆盖本地
git fetch --all
git reset --hard origin/master 
git pull


tag标签
git branch 
git checkout master
git tag v1.0 先切换到master上
git tag 查看所有标签
git log --pretty=oneline --abbrev-commit        找到历史提交的commit id
git tag v0.9 6224937                                        然后打上标签
git show v0.9                                                    按时间顺序列出
git show <tagname>                                        可以看到说明文字
git tag -s v0.2 -m "signed version 0.2 released" fec145a     通过-s用私钥签名一个标签
          签名采用PGP签名，因此，必须首先安装gpg（GnuPG），如果没有找到gpg，或者没有gpg密钥对，就会报错：
          gpg: signing failed: secret key not available
          error: gpg failed to sign the data
          error: unable to sign the tag
命令git show <tagname>可以看到PGP签名信息

git tag -d v0.1     删除
git push origin v1.0     推送某个标签到远程
git push origin --tags          一次性推送全部尚未推送到远程的本地标签
git tag -d v0.9    如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除
git push origin :refs/tags/v0.9     然后，从远程删除。删除命令也是push 


Command line instructions

Git global setup
git config --global user.name "ptadmin"
git config --global user.email "ptadmin@ptmind.com"

Create a new repository
git clone git@gitlab.com:PtmindDev/devops/jumpserver1.3.git
cd jumpserver1.3
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

Existing folder
cd existing_folder
git init
git remote add origin git@gitlab.com:PtmindDev/devops/jumpserver1.3.git
git add .
git commit -m "Initial commit"
git push -u origin master

Existing Git repository
cd existing_repo
git remote rename origin old-origin
git remote add origin git@gitlab.com:PtmindDev/devops/jumpserver1.3.git
git push -u origin --all
git push -u origin --tags
```
<font color="red" size='10'>rollback</font>

已经执行git commit命令了，但是没有push到远程仓库，用以下命令可以回退

git reset --hard HEAD^
上一个版本就是HEAD^,上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。其实这个回退就是将本地的HEAD指针移动到某个版本上而已，所以这个操作是非常快的。

git log    [查看commit记录]
//这个3628164 是一个commit版本号，可以指定回退到那个版本
git reset --hard 3628164 
查看提交记录，也能看到每一次提交的版本号，这是一个很长的哈希值，这个时候我们通过这个版本号可以指定回退到某个版本上，用这个版本号的时候可以只用它前面几位，具体几位看你心情咯，只要能唯一标识这个版本就ok。

git reflog [查看本地会影响HEAD指针的命令操作记录，这个不会同步到远程仓库]
强调一下，注意这里是记录的会影响HEAD指针的操作记录。
简单举例说一下这个git reflog的使用场景。git reflog
假如当前我有三次提交，再假设三次提交的版本号分别为commitnum1,commitnum2,commitnum3,并且我当前处于commitnum3这个版本上(也就是HEAD指向了commitnum3)，这个时候我用命令git reset --hard commitnum2就回退到了comminum2了，好，问题来了，如果突然又想回到commitnum3怎么办呢，当然你可以翻看上去看记录，那个版本号还能看到，但是如果这是第二天了，已经翻不上去了就麻烦了，这个时候git reflog就出场啦，他可以看到你昨天执行git reset --hard commitnum2命令的时候的所在版本号，这个时候就可以用git reset --hard commitnum3来恢复回去。

git_reflog.png

这个结果分三部分，前面黄色的文字就是执行那次命令时所在的版本号，中间的HEAD@[0]是就是HEAD指针变更记录，最后面就是那次命令所做的事情。

最后再啰嗦一点，如果这些commit都还没有同步到远程仓库，你reset后那些commit记录是不会被同步上去的。但是，如果已经同步上去了，就算你本地reset了，虽然本地工作区内容变成了你想要的了，但是记录就抹不去了。

<font color="red" size='10'>10 个很有用的高级 Git 命令</font>

I have been using git for quite some time now and thought of sharing some advanced git commands that you may find useful whether you are working in a team environment or on your personal project.
1. Export changes done in last commit
This command i have been using regularly for sending the changes done to another person for review/integration who is not on git. It will export the recent committed changed files to a zip file.
git archive -o ../updated.zip HEAD $(git diff --name-only HEAD^)
2. Export changed files between two commits
Similarly if you need to export changed files between two commits, you can use this one.
git archive -o ../latest.zip NEW_COMMIT_ID_HERE $(git diff --name-only OLD_COMMIT_ID_HERE NEW_COMMIT_ID_HERE) 
译者信息

迄今，我已经使用Git很长一段时间了，考虑分享一些不管你是团队开发还是个人项目，都受用的高级git命令。
1. 输出最后一次提交的改变
这个命令，我经常使用它 来发送其他没有使用git的人来检查或者集成所修改的。它会输出最近提交的修改内容到一个zip文件中。
git archive -o ../updated.zip HEAD $(git diff --name-only HEAD^)
2. 输出两个提交间的改变
类似的，如果你需要输出某两个提交间的改变时，你可以使用这个。
git archive -o ../latest.zip NEW_COMMIT_ID_HERE $(git diff --name-only OLD_COMMIT_ID_HERE NEW_COMMIT_ID_HERE) 
3. Clone a specific remote branch
If you wish to clone only specific branch from a remote repository without having to clone whole of the repository branches, this will be useful to you.
git initgit remote add -t BRANCH_NAME_HERE -f origin REMOTE_REPO_URL_PATH_HEREgit checkout BRANCH_NAME_HERE
4. Apply patch from Unrelated local repository
If you need to apply a patch from a commit on some other unrelated local repository to your current repository, here is a shortcut way to do that
git --git-dir=PATH_TO_OTHER_REPOSITORY_HERE/.git format-patch -k -1 --stdout COMMIT_HASH_ID_HERE| git am -3 -k
5. Check if your Branch changes are part of Other branch
cherrycommand lets you check whether your branch’s changes are present in some other branch or not. It will display the changes on current branch to given branch and indicate with a + or – sign to indicate if that commit is merged or not. + indicated not present while – indicates present in the given branch. Here is how to do that:
git cherry -v OTHER_BRANCH_NAME_HERE#For example: to check with master branchgit cherry -v master
译者信息

3. 克隆 指定的远程分支
如果你渴望只克隆远程仓库的一个指定分支，而不是整个仓库分支，这对你帮助很大。
git initgit remote add -t BRANCH_NAME_HERE -f origin REMOTE_REPO_URL_PATH_HEREgit checkout BRANCH_NAME_HERE
4. 应用 从不相关的本地仓库来的补丁
如果你需要其它一些不相关的本地仓库作为你现在仓库的补丁，这里就是通往那里的捷径。
git --git-dir=PATH_TO_OTHER_REPOSITORY_HERE/.git format-patch -k -1 --stdout COMMIT_HASH_ID_HERE| git am -3 -k
5. 检测 你的分支的改变是否为其它分支的一部分
cherry命令让我们检测你的分支的改变是否出现在其它一些分支中。它通过+或者-符号来显示从当前分支与所给的分支之间的改变：是否合并了(merged)。.+ 指示没有出现在所给分支中，反之，- 就表示出现在了所给的分支中了。这里就是如何去检测：
git cherry -v OTHER_BRANCH_NAME_HERE#例如: 检测master分支git cherry -v master
6. Start a new Branch with No History
Sometimes you need to start a new branch and do no want to carry the long history along, for example, you want to place the code in public domain(open source) but do no want to share the history.
git checkout --orphan NEW_BRANCH_NAME_HERE
7. Checkout File from Other Branch without Switching Branches
Here is how to fetch just that file you need from other branch without even have to switch branches.
git checkout BRANCH_NAME_HERE -- PATH_TO_FILE_IN_BRANCH_HERE 
译者信息

6.开始一个无历史的新分支
有时，你需要开始一个新分支，但是又不想把很长很长的历史记录带进来，例如，你想在公众区域（开源）放置你的代码，但是又不想别人知道它的历史记录。
git checkout --orphan NEW_BRANCH_NAME_HERE
7. 无切换分支的从其它分支Checkout文件
不想切换分支，但是又想从其它分支中获得你需要的文件：
git checkout BRANCH_NAME_HERE -- PATH_TO_FILE_IN_BRANCH_HERE 
8. Ignore Changes in a Tracked File
If you are working in a team and all of them are working on same branch, chances are you are going to use fetch/merge quite often. but this sometimes resets your environment specific config files which you have to change every time after merge. Using this command, you can ask git to ignore the changes to specific file. So next time you do merge, this file won’t be changed on your system.
git update-index --assume-unchanged PATH_TO_FILE_HERE
9. Check if committed changes are part of a release
The name-rev command can tell you the position of a committ with respect to a last release. Using this you can check if your changes were part of the release or not.
git name-rev --name-only COMMIT_HASH_HERE
译者信息

8.忽略已追踪文件的变动
如果您正在一个团队中工作，而且大家都在同一条branch上面工作，那么您很有可能会经常用到fetch和merge。但是有时候这样会重置您的环境配置文件，如此的话，您就得在每次merge后修改它。使用这一命令，您就能要求git忽视指定文件的变动。这样，下回你再merge的话，这个文件就不会被修改了。
git update-index --assume-unchanged PATH_TO_FILE_HERE
9.检查提交的变动是否是release的一部分
name-rev命令能告诉您一个commit相对于最近一次release的位置。使用这条命令，您就可以检查您所做出的改动是否是release的一部分了。
git name-rev --name-only COMMIT_HASH_HERE
10. Pull with rebase instead of merge
If you are working in a team which is working on same branch, then you have to do fetch/merge or pull quite often. Branch merges in git are recorded with merge commit to indicate when a feature branch was merged with mainstream. But in the scenario of multiple team members working on same branch, the regular merge causes multiple merge messages in the log causing confusion. So you can use rebase with pull to keep the history clear of useless merge messages.
git pull --rebase
Also, you can configure a particular branche to always rebasing:
git config branch.BRANCH_NAME_HERE.rebase true
译者信息

10.使用rebase推送而非merge
如果您正在团队中工作并且整个团队都在同一条branch上面工作，那么您就得经常地进行fetch/merge或者pull。Git中，分支的合并以所提交的merge来记录，以此表明一条feature分支何时与主分支合并。但是在多团队成员共同工作于一条branch的情形中，常规的merge会导致log中出现多条消息，从而产生混淆。因此，您可以在pull的时候使用rebase，以此来减少无用的merge消息，从而保持历史记录的清晰。
git pull --rebase
您也可以将某条branch配置为总是使用rebase推送：
git config branch.BRANCH_NAME_HERE.rebase true

* 我们只需要敲一行命令，告诉Git，以后st就表示status：
* $ git config --global alias.st status
* 好了，现在敲git st看看效果。
* 当然还有别的命令可以简写，很多人都用co表示checkout，ci表示commit，br表示branch：
* $ git config --global alias.co checkout$ git config --global alias.ci commit$ git config --global alias.br branch
* 以后提交就可以简写成：
* $ git ci -m "bala bala bala..."
* --global参数是全局参数，也就是这些命令在这台电脑的所有Git仓库下都有用。
* 在撤销修改一节中，我们知道，命令git reset HEAD file可以把暂存区的修改撤销掉（unstage），重新放回工作区。既然是一个unstage操作，就可以配置一个unstage别名：
* $ git config --global alias.unstage 'reset HEAD'
* 当你敲入命令：
* $ git unstage test.py
* 实际上Git执行的是：
* $ git reset HEAD test.py
* 配置一个git last，让其显示最后一次提交信息：
* $ git config --global alias.last 'log -1'
* 这样，用git last就能显示最近一次的提交：
* $ git lastcommit adca45d317e6d8a4b23f9811c3d7b7f0f180bfe2Merge: bd6ae48 291bea8Author: Michael Liao <askxuefeng@gmail.com>Date:   Thu Aug 22 22:49:22 2013 +0800    merge & fix hello.py
* 甚至还有人丧心病狂地把lg配置成了：
* git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)%Creset' --abbrev-commit"
* 来看看git lg的效果：

* 为什么不早点告诉我？别激动，咱不是为了多记几个英文单词嘛！
* 配置文件
* 配置Git的时候，加上--global是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。
* 配置文件放哪了？每个仓库的Git配置文件都放在.git/config文件中：
* $ cat .git/config [core]    repositoryformatversion = 0    filemode = true    bare = false    logallrefupdates = true    ignorecase = true    precomposeunicode = true[remote "origin"]    url = git@github.com:michaelliao/learngit.git    fetch = +refs/heads/*:refs/remotes/origin/*[branch "master"]    remote = origin    merge = refs/heads/master[alias]    last = log -1
* 别名就在[alias]后面，要删除别名，直接把对应的行删掉即可。
* 而当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中：
* $ cat .gitconfig[alias]    co = checkout    ci = commit    br = branch    st = status[user]    name = YourName    email = your@email.com
* 配置别名也可以直接修改这个文件，如果改错了，可以删掉文件重新通过命令配置。