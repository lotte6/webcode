---
title: Lua Variables
categories: lua
tag: hide
date: 2020-03-16 19:38:49
tags:
---
lua将所有的全局变量/局部变量保存在一个常规table中，这个table一般被称为全局或者某个函数(闭包)的环境。 

为了方便，lua在创建最初的全局环境时，使用全局变量 _G 来引用这个全局环境。因此，在未手工设置环境的情况下，可以使用 _G[varname] 来存取全局变量的值. 

```
for k,v in pairs(_G) do 
  print(k,"->",v) 
end 
```
