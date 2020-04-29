---
title: Macos connect
categories: linux
tag: hide
date: 2020-03-16 17:38:49
tags:
---

macos 主机管理器

```
#!/bin/bash
############################################################################
#    列出目前个人用户下面ssh_config里面的所有主机
#    然后按照列出来的数字，登录相应的服务器
#    配合 .ssh/config 文件使用
############################################################################
var=(`grep -w 'Host' ~/.ssh/config|awk '{print $2}'`)
length=${#var[@]}
serverList=()
n=1
echo '
'
for  i in ${var[@]}
do
        echo "$n  -------------------------->  $i"
        #str="${var[$i]}"
        #serverList=(${serverList[@]} "$str")
        let "n+=1"
done

echo '

'
echo "Please enter number of server you want login: "
#echo ${var[*]}
#echo ${var[0]}
#echo ${var[1]}
#echo ${var[2]}
read serverNum
serverNum=$[serverNum - 1]
ssh_port=`grep -w -A 4 "${var[$serverNum]}" ~/.ssh/config|grep Port|awk '{print $2}'`
echo "Connect server ${var[$serverNum]} $ssh_port"
ssh ${var[$serverNum]} -p $ssh_port
```
