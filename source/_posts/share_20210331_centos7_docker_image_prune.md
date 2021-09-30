---
layout:     post
title:      Docker 清理没有标签、并且不再被容器使用的镜像 
date:       2021-03-31 09:23:21
author:     liangtong
catalog: true
categories: 分享
tags: docker

---



执行以下命令，删除 dangling（没有标签、并且不再被容器使用） images：

```bash
docker image prune
```



