---
title: ELK 定时清理索引
categories: elk
tag: hide
date: 2020-03-16 16:36:49
tags:
---

```
#/bin/bash

ES_URL='http://localhost:9200'
clean_date=$(date -d '-14days' +'%s')
all_indices=`curl -s  http://localhost:9200/_cat/indices |grep -v apm| awk '{print $3}'`
for i in $all_indices;do
        #echo $i
        ctime=`date -d $(echo $i|awk -F_ '{print $NF}'|sed 's/\./-/g') +'%s'`
        #echo $ctime-$clean_date
        if [[ $ctime -le $clean_date ]];then

                curl -XPOST "$ES_URL/$i/_close"
                if [[ ` echo $? ` == 0 ]];then
                        echo -e "$(date '+%Y-%m-%d %H:%M:%S')   索引\e[0;35m$i\e[0m, 关闭完成"
                        curl -XDELETE "$ES_URL/$i"
                        echo -e "$(date '+%Y-%m-%d %H:%M:%S')   索引\e[0;35m$i\e[0m, 删除完成"
                else
                        echo -e "$(date '+%Y-%m-%d %H:%M:%S')   索引\e[0;35m$i\e[0m, 关闭失败，无法进行删除"
                fi
        fi
done

```

