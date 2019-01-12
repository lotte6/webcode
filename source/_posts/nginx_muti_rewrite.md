---
title: nginx的多条件跳转
date: 2018-12-12 18:15:42
tags:
categories: tech
tag: hide
---
```
手机站的一个跳转
set $test $http_user_agent;
if ($test !~* Safari)
{
break;
}

if ($test !~* AppleWebKit)
{
break;
}

if ($test !~* iPhone)
{
 break;
}

if ($test !~* Windows)
{
        rewrite ^(.*)$ http://3g.xs.cn/ break;
}
```