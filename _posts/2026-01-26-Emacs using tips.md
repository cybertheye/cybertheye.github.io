---
title: 🌳记录一下Emacs的批量编辑使用技巧
layout: post
author: cyven
tags: emacs
categories: CS CS::Tech
---


## 插件

- multiple-cursors ([牛逼的操作](http://emacsrocks.com/e13.html))
- region-binding-mode
- expand-region (更通用)
- combobulate (支持语言有限,但是使用的是treesiter)


## 使用场景: 批量编辑

比如现在需要修改代码中的变量名,然后只需要选中这个变量名
执行 `mc/mark-all-like-this` , 就会选中当前 buffer 中所有的这个变量名,此时就可以进行批量编辑

那么选中这个变量名的过程,如果使用 set mark 然后移动,这样效率会很低

```lisp
(defun cyven/mark-symbol-at-point ()
  "选中光标下的整个 symbol，无论光标在其中间还是两端。"
  (interactive)
  (let ((bounds (bounds-of-thing-at-point 'symbol)))
    (if bounds
        (progn
          (goto-char (car bounds))
          (set-mark (cdr bounds))
          (activate-mark))
      (message "No symbol at point"))))


(defun cyven/mark-defun-at-point ()
  (interactive)
  (let ((bounds (bounds-of-thing-at-point 'defun)))
    (if bounds
	(progn
	  (goto-char (car bounds))
	  (set-mark (cdr bounds))
	  (activate-mark))
      (message "No defun at point"))))

```

可以使用这两个方法, 利用内置的 bounds-of-thing-at-point 功能

或者使用第三方插件 expand-region

其实利用embark也可以,但就是步骤稍微多一点

region-binding-mode 可以将快捷键绑定在 一个region选中的时候, 这样就避免了你不知道绑定在哪个全局快捷键

## 技术替代方案

也可以使用像 color-rg, consult-rigpreg 等搜索工具,然后全量替换(embark-export到一个buffer里)
