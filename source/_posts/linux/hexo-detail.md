---
title: hexo 使用
categories: linux
tag: hide
date: 2019-01-16 20:36:49
tags:
---

```
hexo n newpage 新建页面
hexo s 打开测试服务器

hexo c 删除静态页
hexo g 生成静态页
hexo d 部署到线上

D:/page/blog/source/_posts/ 文章存储目录
D:/page/blog/source/img 自己的图片文件存放目录
D:/page/blog/themes/diaspora/source/img 原有默认图片存储位置，剪切到上面目录

首先需要做的：

1、把目录 d:/page/blog/themes/diaspora/source/img 移动到 d:/page/blog/source/img
2、用自己的图片替换原有图片d:/page/blog/source/img


1、新建文章页
hexo n hexo_detail
2、找到文章页,编辑内容
D:/page/blog/source/_posts/hexo_detail.md
3、推送更新
hexo clean && hexo g &&hexo d
```

