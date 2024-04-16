---
title: update cancel & archive anki.org
layout: post
author: cyven
tags: emacs elisp anki
categories: CS CS::Lang CS::Lang::Elisp
---


## 使用`anki-editor`不舒服的地方

因为我现在基本上使用 [anki-editor](https://github.com/louietan/anki-editor) + `org-capture` 来创建 anki 卡片

capture到的`anki.org` org entry 会越来越多

会有两个问题
1. entry数量越来越多，文件也越来越大，不好管理
2. 每次`anki-editor-push-notes`的时候，默认会去update之前所有的卡片，会做一些检查，因为1情况的存在，导致push新卡片的时候就会很慢


因为个人在使用的时候，发现自己其实新的卡片就不会再去作改动了，所以针对以上两个情况

1. archive anki.org
2. 将update的地方做一些改动


## update cancel

原来push note的逻辑在

```lisp
(defun anki-editor--push-note (note)
  "Request AnkiConnect for updating or creating NOTE."
  (if (= (alist-get 'note-id note) -1)
      (anki-editor--create-note note)
    (anki-editor--update-note note)))
```

这里我们添加一个新的自定义变量，直接修改`anki-editor.el`源代码文件

```lisp

(defcustom anki-editor-update-note-flag nil
  "Decide if update or not notes that already pushed. Disable update by default")
```

然后

```lisp
(defun anki-editor--push-note (note)
  "Request AnkiConnect for updating or creating NOTE."
  (if (= (alist-get 'note-id note) -1)
      (anki-editor--create-note note)
    (unless (null anki-editor-update-note-flag)
      (anki-editor--update-note note))))
```
增加一个是否要update的逻辑。

注意，一般Emacs会为了快速加载文件，会将 .el 文件 编译成 .elc 文件
这时候，修改了源代码，需要再 `M-x byte-recompile-file` 编译下，使覆盖之前的 .elc 文件，这样重启Emacs的时候就不会失效了

## archive anki.org文件

```lisp
;; anki archive

(defun anki/archive-all-pushed ()
  (interactive)
  (org-map-entries '(lambda () (when (= (org-outline-level) 1) (anki/do-archive))) nil file))


(defun anki/do-archive ()
  (let* ((pa (org-entry-properties nil "ANKI_NOTE_ID"))
	 (pa-id (cdar pa)))
    (unless (null pa-id)
      (org-archive-subtree-default))))

```

然后在anki.org文件中 执行 `M-x anki/archive-all-pushed` 就行了
