---
title: identify terminfo when using emacsclient in ghostty
layout: post
author: cyven
tags:
categories:
---


## problem

![2026-04-15-17-50-55-screenshoot.png](../assets/img/2026-04-15-17-50-55-screenshoot.png)

environment: Ghostty


## how to fix

### using Emacs.app as server by (server-start)
run

```shell
mkdir ~/.terminfo
( TERMINFO=/Applications/Ghostty.app/Contents/Resources/terminfo infocmp -x xterm-ghostty ) | ( TERMINFO=~/.terminfo tic -x - )
```

### when using `brew service`

modify `.plist`

see [solution](https://github.com/ghostty-org/ghostty/discussions/5902)
