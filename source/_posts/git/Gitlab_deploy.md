---
title: Gitlab docker 部署
categories: git
tag: hide
date: 2020-4-3 12:03:00
tags: 
---


1. 安装私部版 Gitlab，并且使用gitlab cicd，并且安装注册gitlab runner
2. 根据服务器和人员需求优化gitlab 参数，避免出现500 服务器问题情况

```
一、安装docker   
1、服务器执行
yum remove  -y docker \ 
                  docker-client \ 
                  docker-client-latest \ 
                  docker-common \ 
                  docker-latest \ 
                  docker-latest-logrotate \ 
                  docker-logrotate \ 
                  docker-engine 
curl -sSL https://get.docker.com/ | sh 

二、部署gitlab   
1、部署gitlab服务端

docker run --detach \ 
  --hostname gitlab.888b.com \ 
  --publish 81:80 --publish 2222:22 \ 
  --name gitlab \ 
  --restart always \ 
  --volume /data/gitlab/config:/etc/gitlab \ 
  --volume /data/gitlab/logs:/var/log/gitlab \ 
  --volume /data/gitlab/data:/var/opt/gitlab \ 
  gitlab/gitlab-ce:latest

2、修改 socket 或者映射socket目录 
 unicorn['socket'] = '/var/opt/gitlab/socket' 
 puma['socket'] = '/data/gitlab/socket' 
 storage_check['target'] = 'unix:///var/opt/gitlab/socket' 
 
3、让配置文件生效 
docker exec gitlab gitlab-ctl reconfigure 
或者 
docker restart gitlab 
  
3、nginx反向代理 
 
server { 
    listen       80; 
    server_name  gitlab.888d.com; 
    charset utf-8; 
    ssl_session_timeout     10m; 
    ssl_session_cache       shared:SSL:10m; 
    location / { 
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_set_header X-Forwarded-Proto https; 
 
 
        proxy_redirect off; 
        proxy_connect_timeout   240; 
        proxy_send_timeout      240; 
        proxy_read_timeout      240; 
        proxy_pass              http://127.0.0.1:81; 
    } 
} 
```
三、部署gitlab runner(docker安装) 
1、运行客户端
```
docker run --privileged --detach --hostname gitlab.sa123.site --publish 81:80 --publish 22:22  --name gitlab  --restart always  --volume /data/gitlab/config:/etc/gitlab  --volume /data/gitlab/logs:/var/log/gitlab --volume /data/gitlab/data:/var/opt/gitlab  gitlab/gitlab-ce:latest```

2、执行注册 
```
docker exec gitlab-runner gitlab-runner register 
```

四、部署gitlab runner(yum安装)
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh |sudo bash  
yum install -y gitlab-runner 
sed -i 's#/home/gitlab-runner#/data/gitlab-runner#' /etc/systemd/system/gitlab-runner.service 
sed -i 's#"--user" "gitlab-runner"#"--user" "www"#' /etc/systemd/system/gitlab-runner.service 
systemctl daemon-reload && systemctl restart gitlab-runner 
gitlab-runner register
```

五、安装nvm以及下载对应npm版本(使用编译的用户执行以下脚本)
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
source .bashrc
nvm install v12.10.0
```

六、配置gitlab邮箱服务
```
进入容器
docker exec -it gitlab bash
修改gitlab配置文件
vim  /etc/gitlab/gitlab.rb
#添加以下内容
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "mail_service"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "mail_user"
gitlab_rails['smtp_password'] = "mail_passwd"
gitlab_rails['smtp_domain'] = "mai_service"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false

gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'mail_user'
gitlab_rails['gitlab_email_display_name'] = 'Gitlab'

#如果邮箱服务没有开启ssl则取消ssl验证 添加下条内容
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
3、重载配置
gitlab-ctl reconfigure
4、测试发送邮件
gitlab-rails console production
irb(main):001:0> Notify.test_email('收件人邮箱', 'Message Subject', 'Message Body').deliver_now
```




