---
title: git submodule
layout: post
author: cyven
tags:
categories:
---

https://git-scm.com/book/en/v2/Git-Tools-Submodules

## Scenario that git submodule working

> It offten happens while working on one project, you need to use another project from within it.

>  A common issue arises in these scenarios: you want to be able to treat the two projects as separate yet still be able to use one from within the other.

> Suppose you're developing a website and creating Atom feed. Insteed of writing your own Atom-generating code, you want to use the existing library. Usually, there's two way to do that. You have to either include this code from a shared library like Ruby gem, or copy the source code into your own project.

> The issue with including the library is that it's difficult to do some customizing and you have to ensure every client have that library available.

> The issue with copying the code into your own project is that you may have to do the merging job manually when any custom changes encounter the available upstream changes which will be difficult.

## Advantage of using git submodule

Version controll. The compatible issue no longer exist. If you don't like this version, just roll back.


