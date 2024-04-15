---
title: Blog work flow
layout: post
author: cyven
tags: elisp
categories: Tech
---


## 通过交互命令随时创建blog

```lisp

(defun jekyll-insert-date ()
  (interactive)
  (insert (format-time-string "%Y-%m-%d %H:%M:%S")))


(defconst draft-dir "_drafts")
(defconst post-dir "_posts")

(defcustom jekyll-blog-directory "~/Develop/blog/cybertheye.github.io"
  "set the path where your jekyll project based."
  )

(defvar jekyll-drafts-dir (concat jekyll-blog-directory "/" draft-dir))

(defvar jekyll-posts-dir (concat jekyll-blog-directory "/" post-dir))

(defcustom draft-before-post nil
  "create draft first")

(defun jekyll-create-blog (arg title)
  "Jekyll create blog..."
  (interactive "P\nsTitle: ")
  (let* ((date-of-today (format-time-string "%Y-%m-%d"))
	 (file-name (concat date-of-today "-" title ".md"))
	 (draft-flag (if (equal arg '(4)) t draft-before-post)))
    (if draft-flag
	(progn
	  (message "creating file %s in %s" file-name jekyll-drafts-dir)
	  (create-draft file-name))
      (message "creating file %s in %s" file-name jekyll-posts-dir)
      (create-post file-name))))

(defun create-draft (file-name)
  (let ((filename (concat jekyll-drafts-dir "/" file-name)))
    (find-file filename)
    (insert-front-matter file-name)
    ))

(defun create-post (file-name)
  (let ((filename (concat jekyll-posts-dir "/" file-name)))
    (find-file filename)
     (insert-front-matter file-name)))

(defun insert-front-matter (file-name)
  (yas-insert-snippet)
  (beginning-of-buffer)
  (end-of-line)
  (newline)
  (insert "title: " (string-trim-right (car (last (split-string file-name "-"))) ".md"))
  (message "Successfully creating blog...")
  )
```

## 可以从一个entry通过org-export导出md格式然后创建blog

```lisp
(defun jekyll-create-from-org-entry (arg)
  (interactive "P")
  (let ((heading (org-entry-get nil "ITEM")))
    (org-narrow-to-subtree)
    (with-current-buffer (org-md-export-as-markdown)
      (mark-whole-buffer)
      (kill-ring-save 0 0 t)
      (kill-buffer)
      (delete-window))
    (jekyll-create-blog arg heading)
    (goto-char (point-max))
    (newline)
    (yank)))
```


## 转正

如果通过`C-u M-x jekyll-create-blog` 的方法创建出来的是draft也就是草稿，那么需要一个方法方便把它放到post里面

```lisp
(defun jekyll-draft-to-post ()
  (interactive)
  (let* ((filename (buffer-file-name))
	 (newfilename (string-replace draft-dir post-dir filename)))
    (message "%s is moving to %s" filename newfilename)
    (save-buffer)
    (rename-file filename newfilename)
    (kill-buffer (buffer-name))
    (find-file newfilename)))

```


## 支持img
因为`markdown-mode` 对于截图 贴图功能，支持很弱，所以这里自己写了一个


这里我希望的效果就是截图，然后可以直接复制进 .md  文件
原理就是利用 `pngpaste` 库， 通过shell调用这个命令，去将截图粘贴到自定义的 img 文件夹下

```lisp
(defcustom jekyll-img-store-relative-path nil
  "where the image should place that markdown insert.
Most importantly, it SHOULD BE a relative path to the jekyll-blog-directory"
  )

(setq jekyll-img-store-relative-path "assets/img")

(defvar img-execute-method "pngpaste"
  "the software that be used to paste the img. Pngpaste is default.")

(defun jekyll-insert-image ()
  "use pngpaste to do the actual job"
  (interactive)
  ;; make directory , make sure pngpaste is exist, call pngpaste , paste the md url to the point
  (when (null jekyll-img-store-relative-path)
    (user-error "please custom jekyll-img-store-relative-path first"))

  (when (null (executable-find img-execute-method))
    (user-error "please install %s from Homebrew first" img-execute-method))
  ;; name png by time
  (let* ((pngname (format-time-string "%Y-%m-%d-%H-%M-%S-screenshoot.png"))
	 (img-url-path (concat jekyll-img-store-relative-path "/" pngname)) ;;assets/img/xxxx.png
	 (command (concat img-execute-method " " jekyll-blog-directory "/" img-url-path)))
    (shell-command command)
    (insert (format "![%s](../%s)" pngname img-url-path))
    ))

(add-hook 'markdown-mode-hook
	  (lambda ()
	    (local-set-key (kbd "C-c C-y") 'jekyll-insert-image)))

```


可以用，就先这么着吧
