---
title: tool_tmp_key
categories: linux
tag: hide
date: 2018-12-18 21:59:26
tags:
---

创建ssh-agent客户端，添加key做认证，主要用于第三方主机做认证

```
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null
    - ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts