---
title: ⭐️生产环境中docker部署express和nginx服务的网络访问问题
layout: post
author: cyven
tags: docker dev express nginx gateway
categories: CS CS::Tech
---




项目是一个vue开发的网站项目，其中有一个功能是需要通过输入授权码，验证成功，进入内容页面。

在开发环境中，一开始是用`vite.config.ts` 来做代理以及写一个插件来模拟API服务，为了快速测试

然而在生产环境中，vite并不适合做api服务器，需要用express，然后使用nginx做反向代理

一开始，dockerfile,是让nginx和express都在一个docker容器中，

[编写dockerfile]({% post_url 2025-01-23-dockerfile-configure %})



但是要么就是nginx环境有问题，要么就是express有问题，


最后我想还是单一原则好了，一个docker一个服务，所以弄了两个docker，一个运行nginx，一个运行express

但是启动服务，`fetch`请求还是会`502 Gateway`错误，

这时候，有一个原因

因为nginx转发的的地址

```
    location /api/ {
        proxy_pass http://localhost:5001/;
```

而两个服务不在同一个docker，所以其实nginx转发的到`localhost`是在自己容器里面的`localhost`，所以express服务容器是接受不到

只要让两个docker在一个网络就可以了


```
    location /api/ {
    #using express  because docker run -d --name express xxx
        proxy_pass http://express:5001/;
```

`docker run -d -network bridge -p 80:80 -p 443:443 nginx-service`
`docker run -d --name express -network bridge -p 5001:5001 express-service`
