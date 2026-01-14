---
title: 🍎 url parse problem
layout: post
author: cyven
tags: url path
categories: CS CS::Tech
---


## <img src="ox-hugo/xxx.png"> 这个路径是相对路径是吗？是相对谁？

只要不是 `src="www.xxx.com/xxx"`的都是相对路径

没有 / 开头的是相对当前页面
有 / 开头的是相对根域名

比方说我当前页面是 `www.cybertheye.com/braindump/posts/index.html`

对于这个页面中的 html

img 标签
1. `<img src="ox-hugo/xxx.png">` 寻找的是 `www.cybertheye.com/braindump/posts/ox-hugo/xxx.png`
2. `<img src="/ox-hugo/xxx.png">` 寻找的是 `www.cybertheye.com/ox-hugo/xxx.png`
3. `<img src="../ox-hugo/xxx.png">` 寻找的是 `www.cybertheye.com/braindump/ox-hugo/xxx.png`
