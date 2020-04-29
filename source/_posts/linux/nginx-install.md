---
title: nginx编译安装
categories: linux
tag: hide
date: 2018-12-13 19:29:05
tags:
---
安装依赖包
```
apt-get install libcurl4-openssl-dev libpcre3-dev libssl-dev
./configure --user=www-data --group=www-data --prefix=/opt/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre --add-module=../ngx_cache_purge-1.5/
```
配置文件

此配置文件是加了缓存功能，如果无法命中，重新请求一次。
```
user  www-data www-data;
worker_processes 32;
error_log  /var/log/nginx/error.log  crit;

#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 51200;

events
{
    use epoll;
    #maxclient = worker_processes * worker_connections / cpu_number
    worker_connections 2000;
}


http
{
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] $request '
                         '"$status" $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/www.log  main;
    sendfile        on;
    tcp_nopush      on;
    charset gb2312; 
    keepalive_timeout  0;
    server_names_hash_bucket_size 128;
    large_client_header_buffers 4 32k;
    client_max_body_size 8m;
    client_header_buffer_size 32k;
    client_body_buffer_size  512k;
    client_body_temp_path /var/tmp/client_body_temp_path;
    open_file_cache max=65535 inactive=600s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 3;
    send_timeout        10;

    gzip on;
    gzip_min_length 1k;
    gzip_buffers 16 64k;
    gzip_http_version 1.0;
    gzip_comp_level 4;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

   ####fastcgi module#######
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        fastcgi_buffer_size 4k;
        fastcgi_buffers 128 4k;
        fastcgi_busy_buffers_size 128k;
        fastcgi_temp_file_write_size 128k;

        fastcgi_intercept_errors on;
        error_page 404 = /web/notfound.php;

    upstream my_server_pool
        {
        server 121.18.211.10 weight=10 max_fails=2 fail_timeout=30s;
        server 121.18.211.40 weight=10 max_fails=2 fail_timeout=30s;
        server 121.18.211.41 weight=10 max_fails=2 fail_timeout=30s;
        server 121.18.211.46 weight=8 max_fails=2 fail_timeout=30s;
        server 121.18.211.51 weight=8 max_fails=2 fail_timeout=30s;
        server 121.18.211.54 weight=8 max_fails=2 fail_timeout=30s;
        server 121.18.211.55 weight=8 max_fails=2 fail_timeout=30s;
        server 121.18.211.58 weight=10 max_fails=2 fail_timeout=30s;
        }


      server
      {
        listen 80;
        server_name  www.readnovel.com 121.18.211.53;
 location ~ /nginx_status 
  {
                 stub_status on;
   allow 121.18.211.0/24;
                 allow 119.253.57.170;
                }
  
                location ~ (healthcheck.html)$
  {
   root /var/www/nopage;
  }
  
                location ~ .*\.(gif|jpg|png|bmp|swf|js|css)$
  {                       
   proxy_next_upstream http_502 http_504 error timeout invalid_header;
                        proxy_set_header Host 127.0.0.1;
                        proxy_set_header X-Forwarded-For  $remote_addr;
                        proxy_cache cache_two;
                        proxy_cache_valid 200 304 7d;
                        proxy_cache_valid 301 302 5m;
                        proxy_cache_valid any 5m;
                        proxy_cache_key $uri$is_args$args;
                        add_header X-Cache $upstream_cache_status;
                        proxy_set_header X-Forwarded-For $remote_addr;
                        proxy_headers_hash_bucket_size 128;
                        proxy_headers_hash_max_size 512;
                        expires 7d; 
                        proxy_pass http://127.0.0.1;
  }

                location ~ /novel/
                {
   proxy_next_upstream http_502 http_504 error timeout invalid_header;
                 proxy_set_header Host 127.0.0.1; 
                 proxy_set_header X-Forwarded-For  $remote_addr;
                 proxy_cache cache_one;
                 proxy_cache_valid 200 304 48h;
                 proxy_cache_valid 301 302 5m;
                 proxy_cache_valid any 5m;
                 proxy_cache_key $uri$is_args$args;
                 add_header X-Cache $upstream_cache_status;
                 proxy_set_header X-Forwarded-For $remote_addr;
   proxy_headers_hash_bucket_size 128;
   proxy_headers_hash_max_size 512;
   expires 10m;
                 proxy_pass http://127.0.0.1;
  }

  location / {
   proxy_set_header Host 127.0.0.1;
                 proxy_set_header X-Forwarded-For  $remote_addr; 
   proxy_pass http://127.0.0.1;
  }
  
 }
        
 server
        {
         listen 80;
         server_name  nginx.readnovel.com 127.0.0.1;
  root /var/www/readnovel;
  include vhost/*.conf;

                location ~ /purge(/.*)
                {
                  allow all;
                  proxy_cache_purge cache_one $1$is_args$args;
                }
  
  location ~ .*\.php?$
  {
                 fastcgi_pass    127.0.0.1:9000;
                 fastcgi_index   index.php;
                 include         fastcgi.conf;
  }

  
  location ~* .*\.(gif|jpg|png|bmp|swf|js|css)(.*)
  {
          root /var/www/readnovel;
   if (-f $request_filename) 
   {
    expires 7d;
    break;
                 }
  }
  location /
  {
   index index.html ;
                }
  
 

 }
}
```
系统级别的修改
1、修改单个线程打开的最大句柄数

在/etc/security/limits.conf最后增加如下两行记录  
```
* soft nofile 65535  
* hard nofile 65535  
```
到当收到”Too many open files in system”这样的错误消息时 应增加   
```
# echo ""fs.file-max=*****" >> /etc/sysctl.conf  # sysctl -p 
```

2、系统内核参数修改
```
  net.ipv4.ip_local_port_range = 1024 65535   net.ipv4.tcp_syncookies = 1   net.ipv4.tcp_tw_reuse = 1   net.ipv4.tcp_tw_recycle = 1   net.ipv4.tcp_timestamps = 0   net.core.somaxconn =  32768    net.ipv4.tcp_max_orphans = 262144   net.ipv4.tcp_max_syn_backlog = 65536   net.ipv4.tcp_timestamps = 0   net.ipv4.tcp_synack_retries = 2   net.ipv4.tcp_syn_retries = 1 
```
附：

net.core.somaxconn = 32768

web 应用中listen 函数的backlog 默认会给我们内核参数的net.core.somaxconn 限制到128，而nginx 定义的NGX_LISTEN_BACKLOG 默认为511，所以有必要调整这个值。

net.core.netdev_max_backlog = 32768

每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。

net.ipv4.tcp_max_orphans = 262144

系统中最多有多少个TCP 套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，孤儿连接将即刻被复位并打印出警告信息。这个限制仅仅是为了防止简单的DoS 攻

击，不能过分依靠它或者人为地减小这个值，更应该增加这个值(如果增加了内存之后)

net.ipv4.tcp_max_syn_backlog = 65536

记录的那些尚未收到客户端确认信息的连接请求的最大值。对于有128M 内存的系统而言，缺省值是1024，小内存的系统则是128。