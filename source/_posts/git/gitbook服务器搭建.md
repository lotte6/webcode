---
title: Gitbook 部署
categories: git
tag: hide
date: 2020-4-3 12:01:00
tags: 
---

Gitbook: markdown 格式文档，实现文档共享和传承

```
yum install nodejs
npm install -g gitbook-cli
创建gitbook目录
mkdir /data/gitbook
cd /data/gitbook
gitbook init  生成文件
gitbook build 生成静态文件

cd /data/gitbook
touch book.json
在book.json中添加"plugins"和"pluginConfig"字段。然后执行gitbook install

cat book.json
{
    "plugins": [
	"-lunr",
	"-search",
	"search-pro",
        "page-treeview",
        "back-to-top-button",
        "code",
        "chapter-fold",
	"splitter",
	"lightbox",
	"emphasize",
	"pageview-count",
	"tbfed-pagefooter",
	"edit-link",
	"theme-comscore"
    ],
    "pluginsConfig": {
        "page-treeview": {
            "copyright": "Copyright &#169; aleen42",
            "minHeaderCount": "2",
            "minHeaderDeep": "2"
        },
	"edit-link": {
            "base": "http://gitlab.sa123.site/sa/gitbook/blob/master",
            "label": "编辑此页面"
       },
	"theme-default": {
        "showLevel": true
       }
        }
}
gitbook serve   开启服务

nginx配置
server {
    listen       80;
    server_name  wiki.sa123.site;
    charset utf-8;
    location ~*  \.(jpg|jpeg|png|gif|ico|css|js|pdf)$ {
	root /data/gitbook/_book;
        expires 7d;
    auth_basic "请输入密码"; 
    auth_basic_user_file /etc/nginx/passwd;
    }

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
        proxy_connect_timeout   240;
        proxy_send_timeout      240;
        proxy_read_timeout      240;
        proxy_pass              http://10.88.12.115:4000;
    }
}

```
