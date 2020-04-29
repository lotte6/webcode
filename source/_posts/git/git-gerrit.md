---
title: git_gerrit
categories: git
tag: hide
date: 2018-12-18 22:02:28
tags: 
---

<font color="red" size='10'>安装</font>

http://blog.csdn.net/benkaoya/article/details/8680886
1、安装jdk,因为gerrit用java开发的需要，java运行环境，
yum search openjdk
yum install java-1.7.0-openjdk.i686

2、安装git，这个步骤很简单
yum  install git

让git支持中文
git config --global core.quotepath false
git config --global i18n.logoutputencoding utf8
git config --global i18n.commitencoding utf8

3、下载gerrit，我用的是gerrit-2.8.1.war，将文件上传至linux，我放在/usr/local/目录下
cd /usr/local/
wget http://gerrit-releases.storage.googleapis.com/gerrit-2.8.1.war
java -jar gerrit-2..8.1.war init -d gerrit-site
一路回车........


自此，Gerrit安装过程就完成了，可以在windows平台下通过web访问Gerrit（当然也可以直接用虚拟机本地的浏览器访问），即在浏览器中输入http://10.10.1.234:8080。但是需要注意的是：我用的linux是CentOS，需要先关掉防火墙才能连接上。



注意：http://10.10.1.234:8080 要以ip的方式访问，不要用域名访问，用域名的的方式访问，会多一次跳转，跳转到ip形式访问（可能和下面的参数：canonicalWebUrl = http://203.195.187.200:8080/ 有关）

安装 gerrit - m15142436758 - LOWPING的博客

 登录的第一个用户将自动成为管理员（Account ID为1000000的就是管理员），所有后续登录的用户都是无权限用户（需要管理员指定权限）。如果你选择了development_become_any_account，在页面顶端会有一个Become链接，通过它可以进入注册/登录页面。



4、启动，停止服务

cd /usr/local/gerrit-site/bin

./gerrit.sh start

./gerrit.sh stop

./gerrit.sh restart



5、支持ldap验证，email

cd  /usr/local/gerrit-site/etc

vim gerrit.config

支持ldap

修改

[gerrit]

        basePath = git

        canonicalWebUrl = http://203.195.187.200:8080/      经过测试，这里不能写localhost，改成真是ip

[auth]

        type = LDAP          把OPENID 改成LDAP

增加ldap配置

[ldap]

        server = ldap://localhost

        accountBase = ou=People,dc=1v,dc=cn

        groupBase = ou=group,dc=1v,dc=cn

accountFullName = ${cn}



支持email

[sendemail]

        enable = true

        smtpServer = smtp.exmail.qq.com

        smtpUser = luxfang@1v.cn

        smtpPass = ****

        from = Admin<luxfang@1v.cn>

        smtpServerPort = 25

6、配置支持gitweb
1、编辑配置文件，让gerrit能找到gitweb的安装目录就行了
vim /data/gerrit/etc/gerrit.config
添加
[gitweb]
        cgi = /var/www/git/gitweb.cgi
2、重启gerrit

7、升级gerrit
下载更高版本的软件包，覆盖安装就OK了
8、gerrit修改一些东西有些非常麻烦，需要登入gerrit的H2数据库
1、进入gerrit-2.8.1.war所在的目录
java -jar gerrit-2.8.1.war gsql -d /data/gerrit

一般常用的命令
show  tables;
用户一般在  accounts  中
组一般在     account_groups   中

2、删除project
直接到gerrit_home/git/删除对应project就行

9、迁移gerrit

将gerrit从A地迁移到B地
1：在B地安装好与A地相同的版本的gerrit
2：将A地gerrit_home下的git ， db 目录复制到 B地的gerrit_home下

10、gerrit 权限

refs/*   下常用的权限

read         获取整个仓库数据
owner   有这个权限的用户相当于该项目的创建者   

refs/heads/下常用的权限

refs/heads/*                       对所有分支有效
refs/heads/branch_name     对指定分支有效
push    直接提交到某个分支
create  reference 创建分支，或轻量级的tag
Label Code-Review  审核提交（不能再refs/for/*分指上）
Label Code-Review(+2)  + Submit 提交审核通过的代码（不能再refs/for/*分指上）
Submit 提交（不能再refs/for/*分指上）
Push Merge Commits   在master分支上做了merge操作，且要push master时需要的权限（一般用与refs/*搭配，且不包含push权限）
abandon  删除待审核的提交，提交者无须此权限也可操作（不能再refs/for/*分指上）
Remove Reviewer 删除审核列表中的审核成员，提交者无须此权限也可操作
Forge Author 和 Forge Committer 当提交的用户账号和邮箱与服务器不对时，用这2个权限跳过验证
Forge Author 允许他人提交

refs/for/refs/heads/

refs/for/refs/heads/*                        对所有分支有效
refs/for/refs/heads/branch_name     对指定分支有效
push 只能提交到review，不能直接提交

refs/tags/*   下常用的权限
create reference push轻量级标签
Push Annotated Tag  push含附注的标签
Push signed Tag  push含附注的标签
常用配置案例
Reference: refs/*
Read
Push Merge Commit              允许merge操作
Label Code-Review              和submit权限一起组成，决定审核是否通过
Submit

Reference: refs/for/refs/heads/master  master分支上的提交必须经过审核
Push 

Reference: refs/heads/first   first分支上可以直接提交
Push
gerrit权限覆盖

例子1：除了master分支，其他分支可直接提交安装 gerrit - m15142436758 - JoyBank的博客

 例子2：只有master分支可直接提交，其他分支不行


 1 为项目导入群组创建权限
（1）Global Capabilities 中增加 sp_importers 创建项目和群组的权限：
[capability]
        streamEvents = group Non-Interactive Users
+       createProject = group sp_importers
（2）refs/*增加 sp_importers的创建权限：
 [access "refs/*"]
        read = group Administrators
        read = group Anonymous Users
+       create = group sp_importers
2 refs/meta/config增加注册用户的读权限，使 gitweb在其他用户时使用时不会出现not found错误
[access "refs/meta/config"]
    exclusiveGroupPermissions = read
    read = group Administrators
    read = group Project Owners
+    read = group Registered Users
3 refs/heads/*增加注册用户的submit权限，不然即使 Review +2，也无法合并到主线
这里也不用担心+1的其他用户直接 submit，因为只有 Review+2后才能提交上去（一旦review +2，这个提交动作可以由其他用户完成）
[access "refs/heads/*"]
    create = group Administrators
    create = group Project Owners
    forgeAuthor = group Registered Users
    forgeCommitter = group Administrators
    forgeCommitter = group Project Owners
    push = group Administrators
    push = group Project Owners
    label-Code-Review = -2..+2 group Administrators
    label-Code-Review = -2..+2 group Project Owners
    label-Code-Review = -1..+1 group Registered Users
    submit = group Administrators
    submit = group Project Owners
+    submit = group Registered Users





审核使用流程

1、clone 主分支
git clone ssh://gerrit2@127.0.0.1:29418/test_project.git

2、编辑 .gitconfig （家目录下）文件添加个人信息，或者执行命令

git config --global user.name "John Doe"
git config --global user.email johndoe@example.com 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[user]
     name = robert
     email = robert.yang@tech-core.org
[core]
     autocrlf = true
     excludesfile = C:\\Users\\y.y-PC\\Documents\\gitignore_global.txt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

3、项目目录下找到.config文件，编辑如下内容，或者执行命令
git config remote.origin.push refs/heads/*:refs/for/* 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[core]
repositoryformatversion = 0
filemode = false
bare = false
logallrefupdates = true
symlinks = false
ignorecase = true
hideDotFiles = dotGitOnly
[remote "origin"]
url = ssh://robert@gerrit.bazinga.com:29418/test56
fetch = +refs/heads/*:refs/remotes/origin/*
push = refs/heads/*:refs/for/*
[branch "master"]
remote = origin
merge = refs/heads/master
[remote "orgin"]
push = refs/heads/*:refs/for/*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

4、拷贝钩子程序到项目，执行命令
scp -p -P 29418 robert@192.168.169.56:hooks/commit-msg test56/.git/hooks/



提示报错
添加一行
[database]

url = jdbc:mysql://localhost:3306/reviewdb?user=gerrit2&password=<secret>&useUnicode=false&characterEncoding=latin1


<font color="red" size='10'>操作</font>
```
drop database reviewdb;
create database reviewdb DEFAULT CHARACTER SET latin1; 

 java -jar gerrit-full-2.5.2.war init -d gerrit_site_http

ssh -p 29418 admin@127.0.0.1 gerrit create-project -n test 创建项目
ssh -p 29418 admin@127.0.0.1 gerrit ls-projects 查看项目

192.168.169.55
http://192.168.169.55/#/admin/projects/ 
http://192.168.169.55/#/admin/projects/

删除组或者项目
Group
1. access gerrit sql database
   cmd: ssh -p 29418 150.236.40.165 gerrit gsql
2. gerrit> delete from ACCOUNT_GROUP_NAMES where name=<group name>;
3. gerrit> delete from ACCOUNT_GROUPS where name=<group name>;

Project
1. access gerrit sql database
   cmd: ssh -p 29418 150.236.40.165 gerrit gsql
2. gerrit> delete from projects where name=<project name>;
3. gerrit> delete from ref_rights where project_name=<project name>;
4. go to folder /gerrit/review_site/git, remove <project name>.git
```





权限

master分支
项目的master分支默认只有administrators和Project Owners可以不经代码审核直接推送,但是允许其他用户向master分支推送changes接受评审。这里调整为Registered Users组用户不能向master推送changes,而只能向devel分支推送changes。devel分支的权限默认即可。项目只有两个常设分支master和devel,日常开发只在devel分支上,只有管理员才能touch master分支。
Project->list(选定项目)->Access->Edit->Add Reference
reference的名字为:refs/for/refs/heads/master,然后添加push权限,添加组”Registered Users”,选择对应的push权限为deny,同时勾选Exclusive，覆盖掉该ref继承和被通配符所涵盖的权限。

Code Review和submit

gerrit默认只给Registered Users组用户Code View -1分到1分的权限,这样Registered Users组用户就无法独立完成代码审核,而developer都集中在这个组中,因此将其Code View权限调整为-2分到2分。而且改组用户没有submit的权限,无法合并补丁到仓库中,下面一并添加submit权限。

Project->list(选定项目)->Access->Edit->Add Reference

reference的名字为:refs/heads/*,然后添加Label Code-Review,添加组”Registered Users”,将其权限调整为-2 ~ 2。然后再添加Submit权限,添加组”Registered Users”,其权限为ALLOW。

sandbox分支

个人分支还是十分有必要的,在开发成果还没有达到可以参加评审之前,用户可以在个人分支暂存自己的代码。stash暂存区并不能替代个人分支。Gerrit也考虑到了这一点,可以通过配置为每个开发者提供一个独立的区域,可以不用参与代码评审,完全是个人私有的领域。

添加如下引用:
refs/heads/sandbox/${username}/*
然后选择权限Create Reference和push,让”Registered Users”组对应的权限皆为ALLOW就可以了。


权限解释

Abandon
此权限允许用户丢弃一个提交的change。如果用户有push权限，给用户分配此权限的同时用户也被分配了restore a change的权限。

Create Reference
此权限管理用户是有可以创建references，branches，tags。此权限一般与普通的push权限一起被分配。

Forge Author
伪造发起人权限，此权限允许用户绕过提交时的身份验证（Gerrit默认会匹配提交信息中author或者committer行中的email地址，如果 Email地址不匹配，则不允许提交）。

Forge Committer
伪造提交者权限，此权限允许用户绕过提交时的身份验证（Gerrit默认会匹配提交信息中author或者committer行中的email地址，如果 Email地址不匹配，则不允许提交 ）。

Forge Server
伪造Gerrit服务器权限，此权限允许在committer行中使用server owner和email

Owner
此权限允许用户修改香项目的配置，具体如下：

修改项目描述
通过ssh的"create-branch"命令创建分支
在web UI界面创建/删除branch
允许/撤销任何访问权限，包括Owner权限。

Push
此分类控制用户被允许怎样推送新commit到Gerrit。

Direct Push
所有已存在的branch可以快进到新的commit。创建新分支受“Create Reference”控制，不允许删除已存在的分支，这是最安全的模式（因为commit不可以被丢弃）。

Force option
允许已存在的branch被删除。开启此选项可以从项目历史中删除提交记录。
此权限主要用来给那些只想用Gerrit的访问控制，不需要Gerrit的代码审查功能的工程使用。

Upload To Code Review
此push权限分配在refs/for/refs/heads/BRANCH命名空间上，允许用户提交一个未合并(non-merge)的commit到refs/for/BRANCH命名空间，创建一个新的代码审查change。
用户必须能够clone和fetch一个工程才可以提交change，所以用户还必须拥有Read权限。

Push Merge Commit
此权限允许用户提交merge commits，它是Push权限的附属物，如果想只允许通过Gerrit做merge操作，那么应该只分配Push仅限而不分配此权限。

Push Annotated Tag
此类权限允许用户向工程仓库提交一个annotated tag。通常使用以下两种方式提交：

git push ssh://USER@HOST:PORT/PROJECT tag v1.0
或者:
git push https://HOST/PROJECT tag v1.0

Tags必须被注释（使用git tag -a），必须在refs/tags/下存在，而且必须是新的。
一般在工程达到了稳定且可发布的时候会打一个Tag。
此权限允许创建一个未签名的Tag。打Tag者的email地址必须与当前用户的一致。
如果要提交不是自己打的Tag，则必须同时分配Forge Committer Identity权限。
如果要提交轻标签(lightweight tags)分配Create Reference权限给引用/refs/tags/*
如果要删除或覆盖一个已存在的tag，分配Push权限并开启Force option。

Push Signed Tag
此类权限允许用户向工程仓库提交一个PGP签名的 tag。通常使用以下两种方式提交：

git push ssh://USER@HOST:PORT/PROJECT tag v1.0
或者:
git push https://HOST/PROJECT tag v1.0

Tags必须被注释（使用git tag -a），必须在refs/tags/下存在，而且必须是新的。

Read
此类权限控制工程的changes， comments，和code diffs可见性，和是否可通过SSH或HTTP访问Git。
如果在单独工程的ACL中设置的此权限，那么全局ACL中的设置将不起作用。

Rebase
此类仅限允许用户通过web页面的“Rebase Change”按钮衍合（Rebase）修改

Remove Reviewer
此类权限允许用户在一个change的reviewers list中移除其他用户。
change所属者可以移除0分或负分的reviewers（即使没有此权限）。
项目所有者和网站管理员可以移除所有reviewers（即使没有此权限）。
没有此权限的用户只可以移除自己。

Review Labels

Submit
此类权限允许用户提交changes。
提交一个change会使该change尽可能快的合并到目的分支，使其作为项目历史永久的一部分。
为了提交change,所有的labels都必须允许提交，并且不能block它。
如果要快速提交一个push上的change，用户需要在refs/for/<ref>(e.g. on refs/for/refs/heads/master)有此权限。

Submit(On Behalf Of)
此类权限允许有Submit权限的用户代表其他用户提交change。
在project.config文件中，此权限被命名为submitAs。

View Drafts
此类权限允许用户查看其他用户提交的drafts changes
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）

Publish Drafts
此类权限允许用户发布其他用户提交的drafts changes
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）

Delete Drafts
此类权限允许用户删除其他用户提交的drafts changes
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）

Edit Topic Name
允许用户编辑提交到review的change的话题名。
change所用者，分支所用者，项目所用者和网站管理员都可以编辑此话题名（即使没有此权限）。
“Force Edit”标识控制是否可以编辑已关闭的change标题，如果此标识设置只能编辑open changes，则不可以编辑已关闭的change 标题。

Edit Hashtags
允许用户在提交到reviews的changes上添加或移除hashtags。
change所用者和任何明确添加的reviewers也可以查看（即使没用此权限）
