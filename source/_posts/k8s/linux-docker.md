---
title: Docker
categories: k8s
tag: hide
date: 2019-01-04 15:55:03
tags:
---
> Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源
> 容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

# Install

安装 Docker
从 2017 年 3 月开始 docker 在原来的基础上分为两个分支版本: Docker CE 和 Docker EE。

Docker CE 即社区免费版，Docker EE 即企业版，强调安全，但需付费使用。

本文介绍 Docker CE 的安装使用。

移除旧的版本：
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
安装一些必要的系统工具：
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
添加软件源信息：
```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
更新 yum 缓存：
```
sudo yum makecache fast
```
安装 Docker-ce：
```
sudo yum -y install docker-ce
```
启动 Docker 后台服务
```
sudo systemctl start docker
```
测试运行 hello-world
```
[root@runoob ~]# docker run hello-world
```

由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行。

使用脚本安装 Docker
1、使用 sudo 或 root 权限登录 Centos。

2、确保 yum 包更新到最新。
```
$ sudo yum update
```
3、执行 Docker 安装脚本。
```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```
执行这个脚本会添加 docker.repo 源并安装 Docker。

4、启动 Docker 进程。
```
sudo systemctl start docker
```
5、验证 docker 是否安装成功并在容器中执行一个测试的镜像。
```
$ sudo docker run hello-world
docker ps
```
到此，Docker 在 CentOS 系统的安装完成。


# 常用命令

查看容器的root用户密码

docker logs <容器名orID> 2>&1 | grep '^User: ' | tail -n1

因为docker容器启动时的root用户的密码是随机分配的。所以，通过这种方式就可以得到redmine容器的root用户的密码了。


查看容器日志

docker logs -f <容器名orID>

查看正在运行的容器

docker ps
docker ps -a为查看所有的容器，包括已经停止的。

删除所有容器

docker rm $(docker ps -a -q)

删除单个容器

docker rm <容器名orID>

停止、启动、杀死一个容器

docker stop <容器名orID>
docker start <容器名orID>
docker kill <容器名orID>

docker kill $(docker ps -a -q)

查看所有镜像

docker images
删除所有镜像
docker rmi $(docker images | grep none | awk '{print $3}' | sort -r)

运行一个新容器，同时为它命名、端口映射、文件夹映射。以redmine镜像为例

docker run --name redmine -p 9003:80 -p 9023:22 -d -v /var/redmine/files:/redmine/files -v /var/redmine/mysql:/var/lib/mysql sameersbn/redmine

一个容器连接到另一个容器

docker run -i -t --name sonar -d -link mmysql:db   tpires/sonar-server
sonar

容器连接到mmysql容器，并将mmysql容器重命名为db。这样，sonar容器就可以使用db的相关的环境变量了。


拉取镜像

docker pull <镜像名:tag>

如

docker pull sameersbn/redmine:latest

当需要把一台机器上的镜像迁移到另一台机器的时候，需要保存镜像与加载镜像。

机器a

docker save busybox-1 > /home/save.tar

使用scp将save.tar拷到机器b上，然后：

docker load < /home/save.tar

构建自己的镜像

docker build -t <镜像名> <Dockerfile路径>

如Dockerfile在当前路径：

docker build -t xx/gitlab .
重新查看container的stdout

启动top命令，后台运行

$ ID=$(sudo docker run -d ubuntu /usr/bin/top -b)

获取正在running的container的输出

$ sudo docker attach $ID
```
top - 02:05:52 up  3:05,  0 users,  load average: 0.01, 0.02, 0.05
Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.1%us,  0.2%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    373572k total,   355560k used,    18012k free,    27872k buffers
Swap:   786428k total,        0k used,   786428k free,   221740k cached
^C$
$ sudo docker stop $ID
```
后台运行(-d)、并暴露端口(-p)

docker run -d -p 127.0.0.1:33301:22 centos6-ssh

从container中拷贝文件出来

sudo docker cp 7bb0e258aefe:/etc/debian_version .

拷贝7bb0e258aefe中的/etc/debian_version到当前目录下。

注意：只要7bb0e258aefe没有被删除，文件命名空间就还在，可以放心的把exit状态的container的文件拷贝出坑

ubuntu14下的docker是没有service服务。去除每次sudo运行docker命令，需要添加组：
```
# Add the docker group if it doesn't already exist.
$ sudo groupadd docker
#改完后需要重新登陆用户
$ sudo gpasswd -a ${USER} docker
```
ubuntu14的febootstrap没有-i命令

Dockerfile中的EXPOSE、docker run --expose、docker run -p之间的区别
Dockerfile的EXPOSE相当于docker run --expose，提供container之间的端口访问。docker run -p允许container外部主机访问container的端口
```
docker container ls命令用于列出所有容器。
docker container ls [OPTIONS]
名称,简写	默认值	描述
--all, -a	false	显示所有容器(默认只显示运行的)
--filter, -f		根据提供的条件过滤输出
--format		使用Go模板打印容器
--last, -n	-1	显示最后创建的容器(包括所有状态)
--latest, -l	false	显示最新创建的容器(包括所有状态)
--no-trunc	false	不要截断输出
--quiet, -q	false	只显示数字ID
--size, -s	false	显示容器大小
```

# Dockerfile
```
在Dockerfile中用到的命令有  
FROM  
    FROM指定一个基础镜像， 一般情况下一个可用的 Dockerfile一定是 FROM 为第一个指令。至于image则可以是任何合理存在的image镜像。  
    FROM 一定是首个非注释指令 Dockerfile.  
    FROM 可以在一个 Dockerfile 中出现多次，以便于创建混合的images。  
    如果没有指定 tag ，latest 将会被指定为要使用的基础镜像版本。  
MAINTAINER  
    这里是用于指定镜像制作者的信息  
RUN  
    RUN命令将在当前image中执行任意合法命令并提交执行结果。命令执行提交后，就会自动执行Dockerfile中的下一个指令。  
    层级 RUN 指令和生成提交是符合Docker核心理念的做法。它允许像版本控制那样，在任意一个点，对image 镜像进行定制化构建。  
    RUN 指令缓存不会在下个命令执行时自动失效。比如 RUN apt-get dist-upgrade -y 的缓存就可能被用于下一个指令. --no-cache 标志可以被用于强制取消缓存使用。  
ENV  
    ENV指令可以用于为docker容器设置环境变量  
    ENV设置的环境变量，可以使用 docker inspect命令来查看。同时还可以使用docker run --env <key>=<value>来修改环境变量。  
USER  
    USER 用来切换运行属主身份的。Docker 默认是使用 root，但若不需要，建议切换使用者身分，毕竟 root 权限太大了，使用上有安全的风险。  
WORKDIR  
    WORKDIR 用来切换工作目录的。Docker 默认的工作目录是/，只有 RUN 能执行 cd 命令切换目录，而且还只作用在当下下的 RUN，也就是说每一个 RUN 都是独立进行的。如果想让其他指令在指定的目录下执行，就得靠 WORKDIR。WORKDIR 动作的目录改变是持久的，不用每个指令前都使用一次 WORKDIR。  
COPY  
    COPY 将文件从路径 <src> 复制添加到容器内部路径 <dest>。  
    <src> 必须是想对于源文件夹的一个文件或目录，也可以是一个远程的url，<dest> 是目标容器中的绝对路径。  
    所有的新文件和文件夹都会创建UID 和 GID 。事实上如果 <src> 是一个远程文件URL，那么目标文件的权限将会是600。  
ADD  
    ADD 将文件从路径 <src> 复制添加到容器内部路径 <dest>。  
    <src> 必须是想对于源文件夹的一个文件或目录，也可以是一个远程的url。<dest> 是目标容器中的绝对路径。  
    所有的新文件和文件夹都会创建UID 和 GID。事实上如果 <src> 是一个远程文件URL，那么目标文件的权限将会是600。  
VOLUME  
    创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。  
EXPOSE  
    EXPOSE 指令指定在docker允许时指定的端口进行转发。  
  
CMD  
    Dockerfile.中只能有一个CMD指令。 如果你指定了多个，那么最后个CMD指令是生效的。  
    CMD指令的主要作用是提供默认的执行容器。这些默认值可以包括可执行文件，也可以省略可执行文件。  
    当你使用shell或exec格式时，  CMD 会自动执行这个命令。  
ONBUILD  
    ONBUILD 的作用就是让指令延迟執行，延迟到下一个使用 FROM 的 Dockerfile 在建立 image 时执行，只限延迟一次。  
    ONBUILD 的使用情景是在建立镜像时取得最新的源码 (搭配 RUN) 与限定系统框架。  
ARG  
    ARG是Docker1.9 版本才新加入的指令。  
    ARG 定义的变量只在建立 image 时有效，建立完成后变量就失效消失  
LABEL  
    定义一个 image 标签 Owner，并赋值，其值为变量 Name 的值。(LABEL Owner=$Name )  
  
ENTRYPOINT  
    是指定 Docker image 运行成 instance (也就是 Docker container) 时，要执行的命令或者文件。 
```

# docker 管理
```
镜像管理
docker images：列出本地所有镜像
docker search <IMAGE_ID/NAME>：查找image
docker pull <IMAGE_ID>：下载image
docker push <IMAGE_ID>：上传image
docker rmi <IMAGE_ID>：删除image

容器管理
docker run -i -t <IMAGE_ID> /bin/bash：-i：标准输入给容器    -t：分配一个虚拟终端    /bin/bash：执行bash脚本
-d：以守护进程方式运行（后台）
-P：默认匹配docker容器的5000端口号到宿主机的49153 to 65535端口
-p <HOT_PORT>:<CONTAINER_PORT>：指定端口号
- -name： 指定容器的名称
- -rm：退出时删除容器

docker stop <CONTAINER_ID>：停止container
docker start <CONTAINER_ID>：重新启动container
docker ps - Lists containers.
-l：显示最后启动的容器
-a：同时显示停止的容器，默认只显示启动状态

docker attach <CONTAINER_ID> 连接到启动的容器
docker logs <CONTAINER_ID>  : 输出容器日志
-f：实时输出
docker cp <CONTAINER_ID>:path hostpath：复制容器内的文件到宿主机目录上
docker rm <CONTAINER_ID>：删除container
docker rm `docker ps -a -q`：删除所有容器
docker kill `docker ps -q`
docker rmi `docker images -q -a`
docker wait <CONTAINER_ID>：阻塞对容器的其他调用方法，直到容器停止后退出

docker top <CONTAINER_ID>：查看容器中运行的进程
docker diff <CONTAINER_ID>：查看容器中的变化
docker inspect <CONTAINER_ID>：查看容器详细信息（输出为Json）
-f：查找特定信息，如docker inspect -f '{{ .NetworkSettings.IPAddress }}'
      docker commit -m "comment" -a "author" <CONTAINER_ID>  ouruser/imagename:tag

      docker extc -it <CONTAINER> <COMMAND>：在容器里执行命令，并输出结果


网络管理
docker run -P：随机分配端口号
docker run -p 5000:5000：绑定特定端口号（主机的所有网络接口的5000端口均绑定容器的5000端口）
docker run -p 127.0.0.1:5000:5000：绑定主机的特定接口的端口号
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py：绑定udp端口号
docker port <CONTAINER_ID> 5000：查看容器的5000端口对应本地机器的IP和端口号
使用Docker Linking连接容器：
Docker为源容器和接收容器创建一个安全的通道，容器之间不需要暴露端口，接收的容器可以访问源容器的数据
docker run -d -P --name <CONTAINER_NAME> --link <CONTAINER_NAME_TO_LINK>:<ALIAS>  

数据管理
Data Volumes：volume是在一个或多个容器里指定的特殊目录
数据卷可以在容器间共享和重复使用
可以直接修改容器卷的数据
容器卷里的数据不会被包含到镜像中
容器卷保持到没有容器再使用它
可以在容器启动的时候添加-v参数指定容器卷，也可以在Dockerfile里用VOLUMN命令添加
docker run -d -P --name web -v /webapp training/webapp python app.py
也可以将容器卷挂载到宿主机目录或宿主机的文件上，<容器目录或文件>的内容会被替换为<宿主机目录或文件>的内容，默认容器对这个目录有可读写权限
docker run -d -P --name web -v <宿主机目录>:<容器目录> training/webapp python app.py
可以通过指定ro，将权限改为只读
docker run -d -P --name web -v <宿主机目录>:<容器目录>:ro training/webapp python app.py
在一个容器创建容器卷后，其他容器便可以通过--volumes-from共享这个容器卷数据，如下：
docker run -d -v /dbdata --name db1 training/postgres echo Data-only container for postgres
首先启动了一个容器，并为这个容器增加一个数据卷/dbdata，然后启动另一个容器，共享这个数据卷
docker run -d --volumes-from db1 --name db2 training/postgres
此时db2使用了db1的容器卷，当容器db1被删除时，容器卷也不会被删除，只有所有容器不再使用此容器卷时，才会被删除
docker rm -v：删除容器卷
除了共享数据外，容器卷另一个作用是用来备份、恢复和迁移数据
docker run --volumes-from db1 -v /home/backup:/backup ubuntu tar cvf /backup/backup.tar /dbdata
启动一个容器数据卷使用db1容器的数据卷，同时新建立一个数据卷指向宿主机目录/home/backup，将/dbdata目录的数据压缩为/backup/backup.tar
docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
docker run --volumes-from dbdata2 -v /home/backup:/backup busybox tar xvf /backup/backup.tar
启动一个容器，同时把backup.tar的内容解压到容器的backup

仓库管理
docker login：登录
```


# 常见问题
## 容器报错无法重启解决
docker commit <container_id> frank/ocserv
docker rm ocserv
docker run -it  --name ocserv --privileged -p 443:443 -p 443:443/udp -d frank/ocserv

## 在容器里面修改用户密码的时候报错：
 /usr/share/cracklib/pw_dict.pwd: No such file or directory
 PWOpen: No such file or directory
 
解决：
 yum -y reinstall cracklib-dicts



#  常用命令
## 查看容器的root用户密码

docker logs <容器名orID> 2>&1 | grep '^User: ' | tail -n1

因为docker容器启动时的root用户的密码是随机分配的。所以，通过这种方式就可以得到redmine容器的root用户的密码了。


## 查看容器日志

docker logs -f <容器名orID>

## 查看正在运行的容器

docker ps
docker ps -a为查看所有的容器，包括已经停止的。

## 删除所有容器

docker rm $(docker ps -a -q)

## 删除单个容器

docker rm <容器名orID>

## 停止、启动、杀死一个容器

docker stop <容器名orID>
docker start <容器名orID>
docker kill <容器名orID>

docker kill $(docker ps -a -q)

## 查看所有镜像

docker images
## 删除所有镜像
docker rmi $(docker images | grep none | awk '{print $3}' | sort -r)

## 运行一个新容器，同时为它命名、端口映射、文件夹映射。以redmine镜像为例

docker run --name redmine -p 9003:80 -p 9023:22 -d -v /var/redmine/files:/redmine/files -v /var/redmine/mysql:/var/lib/mysql sameersbn/redmine

## 一个容器连接到另一个容器

docker run -i -t --name sonar -d -link mmysql:db   tpires/sonar-server
sonar

容器连接到mmysql容器，并将mmysql容器重命名为db。这样，sonar容器就可以使用db的相关的环境变量了。


## 拉取镜像

docker pull <镜像名:tag>

如

docker pull sameersbn/redmine:latest

当需要把一台机器上的镜像迁移到另一台机器的时候，需要保存镜像与加载镜像。

机器a

docker save busybox-1 > /home/save.tar

使用scp将save.tar拷到机器b上，然后：

docker load < /home/save.tar

## 构建自己的镜像
```
docker build -t <镜像名> <Dockerfile路径>

如Dockerfile在当前路径：

docker build -t xx/gitlab .
重新查看container的stdout
# 启动top命令，后台运行
$ ID=$(sudo docker run -d ubuntu /usr/bin/top -b)
# 获取正在running的container的输出
$ sudo docker attach $ID
top - 02:05:52 up  3:05,  0 users,  load average: 0.01, 0.02, 0.05
Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.1%us,  0.2%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    373572k total,   355560k used,    18012k free,    27872k buffers
Swap:   786428k total,        0k used,   786428k free,   221740k cached
^C$
$ sudo docker stop $ID

后台运行(-d)、并暴露端口(-p)

docker run -d -p 127.0.0.1:33301:22 centos6-ssh

从container中拷贝文件出来

sudo docker cp 7bb0e258aefe:/etc/debian_version .

拷贝7bb0e258aefe中的/etc/debian_version到当前目录下。

注意：只要7bb0e258aefe没有被删除，文件命名空间就还在，可以放心的把exit状态的container的文件拷贝出来

坑

ubuntu14下的docker是没有service服务。去除每次sudo运行docker命令，需要添加组：

# Add the docker group if it doesn't already exist.
$ sudo groupadd docker
#改完后需要重新登陆用户
$ sudo gpasswd -a ${USER} docker

ubuntu14的febootstrap没有-i命令

Dockerfile中的EXPOSE、docker run --expose、docker run -p之间的区别
Dockerfile的EXPOSE相当于docker run --expose，提供container之间的端口访问。docker run -p允许container外部主机访问container的端口

docker container ls命令用于列出所有容器。
docker container ls [OPTIONS]

名称,简写	默认值	描述
--all, -a	false	显示所有容器(默认只显示运行的)
--filter, -f		根据提供的条件过滤输出
--format		使用Go模板打印容器
--last, -n	-1	显示最后创建的容器(包括所有状态)
--latest, -l	false	显示最新创建的容器(包括所有状态)
--no-trunc	false	不要截断输出
--quiet, -q	false	只显示数字ID
--size, -s	false	显示容器大小
```
## 镜像管理
```
docker images：列出本地所有镜像
docker search <IMAGE_ID/NAME>：查找image
docker pull <IMAGE_ID>：下载image
docker push <IMAGE_ID>：上传image
docker rmi <IMAGE_ID>：删除image
```
## 容器管理
```
docker run -i -t <IMAGE_ID> /bin/bash：-i：标准输入给容器    -t：分配一个虚拟终端    /bin/bash：执行bash脚本
-d：以守护进程方式运行（后台）
-P：默认匹配docker容器的5000端口号到宿主机的49153 to 65535端口
-p <HOT_PORT>:<CONTAINER_PORT>：指定端口号
- -name： 指定容器的名称
- -rm：退出时删除容器

docker stop <CONTAINER_ID>：停止container
docker start <CONTAINER_ID>：重新启动container
docker ps - Lists containers.
-l：显示最后启动的容器
-a：同时显示停止的容器，默认只显示启动状态

docker attach <CONTAINER_ID> 连接到启动的容器
docker logs <CONTAINER_ID>  : 输出容器日志
-f：实时输出
docker cp <CONTAINER_ID>:path hostpath：复制容器内的文件到宿主机目录上
docker rm <CONTAINER_ID>：删除container
docker rm `docker ps -a -q`：删除所有容器
docker kill `docker ps -q`
docker rmi `docker images -q -a`
docker wait <CONTAINER_ID>：阻塞对容器的其他调用方法，直到容器停止后退出

docker top <CONTAINER_ID>：查看容器中运行的进程
docker diff <CONTAINER_ID>：查看容器中的变化
docker inspect <CONTAINER_ID>：查看容器详细信息（输出为Json）
-f：查找特定信息，如docker inspect -f '{{ .NetworkSettings.IPAddress }}'
      docker commit -m "comment" -a "author" <CONTAINER_ID>  ouruser/imagename:tag

      docker extc -it <CONTAINER> <COMMAND>：在容器里执行命令，并输出结果
```

## 网络管理
```
docker run -P：随机分配端口号
docker run -p 5000:5000：绑定特定端口号（主机的所有网络接口的5000端口均绑定容器的5000端口）
docker run -p 127.0.0.1:5000:5000：绑定主机的特定接口的端口号
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py：绑定udp端口号
docker port <CONTAINER_ID> 5000：查看容器的5000端口对应本地机器的IP和端口号
使用Docker Linking连接容器：
Docker为源容器和接收容器创建一个安全的通道，容器之间不需要暴露端口，接收的容器可以访问源容器的数据
docker run -d -P --name <CONTAINER_NAME> --link <CONTAINER_NAME_TO_LINK>:<ALIAS>  
```
## 数据管理
```
Data Volumes：volume是在一个或多个容器里指定的特殊目录
数据卷可以在容器间共享和重复使用
可以直接修改容器卷的数据
容器卷里的数据不会被包含到镜像中
容器卷保持到没有容器再使用它
可以在容器启动的时候添加-v参数指定容器卷，也可以在Dockerfile里用VOLUMN命令添加
docker run -d -P --name web -v /webapp training/webapp python app.py
也可以将容器卷挂载到宿主机目录或宿主机的文件上，<容器目录或文件>的内容会被替换为<宿主机目录或文件>的内容，默认容器对这个目录有可读写权限
docker run -d -P --name web -v <宿主机目录>:<容器目录> training/webapp python app.py
可以通过指定ro，将权限改为只读
docker run -d -P --name web -v <宿主机目录>:<容器目录>:ro training/webapp python app.py
在一个容器创建容器卷后，其他容器便可以通过--volumes-from共享这个容器卷数据，如下：
docker run -d -v /dbdata --name db1 training/postgres echo Data-only container for postgres
首先启动了一个容器，并为这个容器增加一个数据卷/dbdata，然后启动另一个容器，共享这个数据卷
docker run -d --volumes-from db1 --name db2 training/postgres
此时db2使用了db1的容器卷，当容器db1被删除时，容器卷也不会被删除，只有所有容器不再使用此容器卷时，才会被删除
docker rm -v：删除容器卷
除了共享数据外，容器卷另一个作用是用来备份、恢复和迁移数据
docker run --volumes-from db1 -v /home/backup:/backup ubuntu tar cvf /backup/backup.tar /dbdata
启动一个容器数据卷使用db1容器的数据卷，同时新建立一个数据卷指向宿主机目录/home/backup，将/dbdata目录的数据压缩为/backup/backup.tar
docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
docker run --volumes-from dbdata2 -v /home/backup:/backup busybox tar xvf /backup/backup.tar
启动一个容器，同时把backup.tar的内容解压到容器的backup
```
## 仓库管理
```
docker login：登录
```
