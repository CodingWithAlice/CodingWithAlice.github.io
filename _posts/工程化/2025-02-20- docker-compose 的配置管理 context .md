---
layout:     post
title:     docker-compose 的配置管理 context 
subtitle:  
date:       2025-02-20
author:     
header-img: 
catalog: true
tags:
    - < 工作经验沉淀 >
typora-root-url: ..
---



# docker-compose 的配置管理 context 

context ：主要用于指定 Docker 构建上下文的路径

- 这个路径可以是本地文件系统上的一个目录，也可以是一个 Git 仓库的 URL

```js
// 修改前
context: ./package/server //  指定构建上下文为当前目录下的 package/server 目录
dockerfile: dockerfile
```

- `dockerfile` 中的 `COPY` 和 `ADD` 指令用于将构建上下文中的文件复制到镜像中，`context` 的配置会影响这些指令能够访问的文件范围

```js
// 修改后
context: . // 不仅要用到 ./package/server 目录下的文件
dockerfile: package/server/dockerfile
```
