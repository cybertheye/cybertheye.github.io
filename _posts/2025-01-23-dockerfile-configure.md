---
title: 🧀 dockerfile 配置问题
layout: post
author: cyven
tags: docker dockerfile
categories: CS CS::Tech
---



# FROM

```
FROM nginx:alpine
FROM express:23-alpine
```

如果这样写`FROM`语句，会有问题，因为这里涉及到一个多级构建的概念

首先`FROM`的概念，

> The FROM instruction initializes a new build stage and sets the base image for subsequent instructions

多个FROM，最后仅会使用最终阶段构建的文件，如果希望这个镜像最终既有nginx,也有express环境的话，那么这么写FROM就会出错

# CMD

```
CMD ["nginx", "-g", "daemon off;"]
CMD ["node", "/server/index.js"]
```

这样也有问题，

> There can only be one CMD instruction in a Dockerfile. If you list more than one CMD, only the last one takes effect.


