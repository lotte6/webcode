---
title: alpine 发行版基础工具
categories: linux
tag: hide
date: 2020-03-16 16:38:49
tags:
---

更换源
使用阿里镜像 https://mirrors.aliyun.com
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

使用科大镜像 http://mirrors.ustc.edu.cn
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

修改 composer 的全局配置文件
composer config -g repo.packagist composer https://packagist.phpcomposer.com

基础管理工具
apk add openrc openssh-server openssh vim git  openntpd logrotate  curl


apk add nginx  libpng-dev php7-zlib libzip-dev jpeg-dev freetype-dev \
&& composer php7 php7-fpm php7-opcache php7-gd php7-mysqli php7-zlib php7-curl


php:7.2-fpm

rc-update add nginx default
rc-update add php-fpm7 default

docker-php-ext-install -j 2 zip gd mysqli pdo pdo_mysql 
docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && docker-php-ext-install -j$(nproc) gd


 apk --update --repository=http://dl-4.alpinelinux.org/alpine/edge/testing add \
    php7 \
    php7-common \
    php7-intl \
    php7-gd \
    php7-mcrypt \
    php7-openssl \
    php7-gmp \
    php7-json \
    php7-dom \
    php7-pdo \
    php7-zip \
    php7-zlib \
    php7-mysqli \
    php7-bcmath \
    php7-pdo_mysql \
    php7-gettext \
    php7-xmlreader \
    php7-xmlrpc \
    php7-bz2 \
    php7-iconv \
    php7-curl \
    php7-ctype \
    php7-fpm \
    php7-mbstring \
    php7-session \
    php7-phar


##### 服务管理工具
```
Type the following command:
# rc-status

Runlevel: default
 crond                                  [  started  ]
 networking                             [  started  ]
Dynamic Runlevel: hotplugged
Dynamic Runlevel: needed/wanted
Dynamic Runlevel: manual
The default run level is called default, and it started crond and networking service for us.

View service list
Type the following command:
# rc-status --list

Sample outputs:

boot
nonetwork
default
sysinit
shutdown
You can change run level using the rc command:
# rc {runlevel}
# rc boot
# rc default
# rc shutdown

boot – Generally the only services you should add to the boot runlevel are those which deal with the mounting of filesystems, set the initial state of attached peripherals and logging. Hotplugged services are added to the boot runlevel by the system. All services in the boot and sysinit runlevels are automatically included in all other runlevels except for those listed here.
single – Stops all services except for those in the sysinit runlevel.
reboot – Changes to the shutdown runlevel and then reboots the host.
shutdown – Changes to the shutdown runlevel and then halts the host.
default – Used if no runlevel is specified. (This is generally the runlevel you want to add services to.)
To see manually started services, run:
# rc-status --manual
apache2

To see crashed services, run:
# rc-status --crashed

How to list all available services
Type the following command:
# rc-service --list
# rc-service --list | grep -i nginx

If apache2/nginx not installed, try the apk command to install it:
# apk add apache2

How to add/enable service at boot time
The syntax is:
rc-update add {service-name} {run-level-name}

To add apache2 service at boot time, run:
# rc-update add apache2

OR
# rc-update add apache2 default

Sample outputs:

 * service apache2 added to runlevel default
How to start/stop/restart services on Alpine Linux
The syntax is as as follows:

How to start service
The syntax is:
# rc-service {service-name} start

OR
# /etc/init.d/{service-name} start

How to stop service
The syntax is:
# rc-service {service-name} stop

OR
# /etc/init.d/{service-name} stop

How to restart service
The syntax is:
# rc-service {service-name} restart

OR
# /etc/init.d/{service-name} restart

Thus to stop, start, and restart an Apache2 service:
# rc-service apache2 stop
# rc-service apache2 start
### [ edit config file ] ###
# vi /etc/apache2/httpd.conf
### [ restart apache 2 after editing the file ] ###
# rc-service apache2 restart
```