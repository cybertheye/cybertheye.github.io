---
title: 🌳create c/cpp project script in elisp
layout: post
author: cyven
tags: elisp emacs
categories: CS CS::Tech
---


# 为 Emacs 打造高效 C/C++ 项目脚手架

## 采用了**迭代式开发（Iterative Development）**，分步构建功能

### 阶段一：建立物理骨架

第一步是解决“有无”问题。使用 make-directory 和 with-temp-file 快速生成目录和初始文件。

```lisp
;; 核心逻辑：使用 expand-file-name 处理路径，避免拼接隐患
(let* ((project-path (expand-file-name project-name parent-path))
       (src-path (expand-file-name "src" project-path)))
  (make-directory src-path t)
  ;; 写入基础的 main.c 和 CMakeLists.txt
  ...)
```

### 阶段二：环境自动化（解决跳转的关键）

这是最重要的一步。在生成文件后，脚本自动执行 Shell 命令，生成 clangd 必需的索引文件。

- Git 初始化：git init

- CMake 配置：cmake -S . -B build -G Ninja (生成编译数据库)

- 创建软链接：将 build/compile_commands.json 映射到根目录，确保 lsp-bridge 开箱即用。


### 阶段三：模板化与交互增强

为了让工具更灵活，我们加入了模板支持。

- 默认逻辑：优先读取本地 templates/ 目录下的 .tpl 文件。

- 交互增强：利用 (interactive "P...") 捕获 C-u 前缀参数。当按下 C-u 时，允许用户手动选择特定的模板文件。


## 核心代码实现

```lisp
(defvar cyven/c-cpp-templates-dir "~/.emacs.d/templates/" "模板存放目录")

(defun cyven/init-c-cpp-project (arg parent-path project-name)
  "自动化创建 C/C++ 项目并配置 LSP 环境。"
  (interactive "P\nD父目录: \ns项目名称: ")
  (let* ((project-path (expand-file-name project-name parent-path))
         (main-file-path (expand-file-name "main.c" project-path))
         (cmake-file-path (expand-file-name "CMakeLists.txt" project-path)))

    ;; 1. 创建目录结构
    (make-directory (expand-file-name "src" project-path) t)
    (make-directory (expand-file-name "include" project-path) t)

    ;; 2. 处理模板逻辑 (支持 C-u 手动选择)
    (let ((main-tpl (expand-file-name "main.c.tpl" cyven/c-cpp-templates-dir))
          (cmake-tpl (if (equal arg '(4))
                         (read-file-name "选择 CMake 模板: " cyven/c-cpp-templates-dir)
                       (expand-file-name "CMakeLists.txt.tpl" cyven/c-cpp-templates-dir))))

      ;; 写入文件内容... (代码略)

      ;; 3. 环境自动化
      (let ((default-directory project-path))
        (shell-command "git init")
        (shell-command "cmake -S . -B build -G Ninja")
        (shell-command "ln -sf build/compile_commands.json .")) ; 强制覆盖链接

      ;; 4. 极致体验：自动打开 main.c
      (find-file main-file-path))))

```

## 技术反思与总结
为什么不直接用 insert-file-contents？
在开发过程中，“直接插入文件内容”与“读取为字符串再插入”的区别:

- 直接插入：简单、快，适合静态模板。

- 字符串缓冲：适合需要二次加工的情况（例如用 format 替换模板中的项目名称占位符）。

慢即是快 (Slow is Smooth, Smooth is Fast)
起初我觉得编写这个脚本很慢，甚至逻辑会“打死节”。但通过：

- 分治法：先写逻辑，再写语法。

- 伪代码先行：理清数据流向。

- 小步迭代：每跑通一个功能再写下一个。

最终不仅得到了一个好用的工具，更深刻理解了 Emacs Buffer 与 File 之间的底层交互逻辑。
