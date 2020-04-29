---
title: ssh-agent 使用
categories: linux
tag: hide
date: 2019-01-22 12:03:44
tags:
---
> ssh-agent是一种控制用来保存公钥身份验证所使用的私钥的程序。
> ssh-agent是一个密钥管理器，运行ssh-agent以后，使用ssh-add将私钥交给ssh-agent保管，其他程序需要身份验证的时候可以将验证申请交给ssh-agent来完成整个认证过程。

### 优势

1. 不用重复输入密码。
2. 不用到处部署私钥




```
- eval $(ssh-agent -s)
- echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null
- ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
- chmod 644 ~/.ssh/known_hosts
```


假设私钥分别可以登录同一内网的主机 A 和主机 B，出于一些原因，不能直接登录 B。可以通过在 A 上部署私钥或者设置 Forwarding 登录 B，也可以转发认证代理连接在 A 上面使用ssh-agent私钥登录 B；可以在A上直接sftp传文件到B上。

如这边有一台机器是local，能通过公钥直接登陆server1和server2。server1和server2之间无公钥登陆。

现在要在server1上直接登陆server2，在local上执行

ssh-agent
ssh-add

执行了之后就将私钥给ssh-agent保管了，server1上面登陆server2的时候，需要私钥验证的时候直接找ssh-agent要就可以了。

接下来登陆server1，注意-A

ssh -A server1
可以发现server1上多了/tmp/ssh-xxxxxxxxx/agent.xxxxx的socket，之后神奇的事发了，在 server1上可直接进server2，只需执行如下命令，如果加了-A则可以继续ssh forwarding，以至无限的机器forwarding。

ssh (-A) server2
同样的原理可以试一下sftp, scp等基于ssh的命令。

如运行ssh-add，遇到“Could not open a connection to your authentication agent.”。

解决：需要ssh-agent启动bash，或者说把bash挂到ssh-agent下面。

ssh-agent bash --login -i
ssh-add