---
title: Emacs 搜索与批量编辑：彻底搞懂 Occur-mode 与 Grep-mode 的区别
layout: post
author: cyven
tags: emacs wgrep ripgrep productivity
categories: CS CS::Tech
---




在 Emacs 中，`consult` + `embark` + `wgrep` 的组合被誉为“搜索与重构的神器”。但很多开发者在初学时都会遇到一个坑：**为什么我导出的搜索结果 Buffer 是只读的？为什么 `wgrep` 报错说模式不对？**

这背后的核心在于：你没有分清 **Occur-mode** 和 **Grep-mode**。

## 一、 问题的起因

假设你正在使用 `consult-line` 搜索当前文件中的某个变量，然后通过 `embark-export` 导出了一个结果 Buffer，准备大干一场进行批量修改。

你习惯性地执行 `M-x wgrep-change-to-wgrep-mode`，却发现 Buffer 依然是只读，或者没有任何反应。

**真相是：** `consult-line` 导出的结果是 `occur-mode`，而 `wgrep` 插件是专为 `grep-mode` 设计的。它们就像是两把钥匙，对不上号。

---

## 二、 Occur-mode vs. Grep-mode：本质区别

了解这两个模式的区别，是进阶 Emacs 搜索专家的必经之路。

| 特性 | **Occur-mode** | **Grep-mode** |
| :--- | :--- | :--- |
| **数据来源** | **内存（Buffer）** | **硬盘（Files）** |
| **常用命令** | `consult-line`, `occur` | `consult-ripgrep`, `grep`, `ag` |
| **搜索范围** | 当前已打开的 Buffer | 指定目录或整个项目的文件 |
| **编辑工具** | 内置的 `occur-edit-mode` | 第三方插件 `wgrep` |
| **同步方式** | `C-c C-c` 保存回原 Buffer | `C-c C-c` 写入磁盘文件 |

### 1. Occur-mode：Buffer 的缩影
当你搜的是“当前文件”时，Emacs 认为数据已经在内存里了。`Occur` 模式只是把匹配的行摘出来放在一个新 Buffer 里。因为它处理的是内存中的文本，所以它**自带编辑功能**。
*   **如何编辑：** 在导出 Buffer 中按 **`e`** (或 `M-x occur-edit-mode`)。
*   **如何保存：** 修改完后按 **`C-c C-c`**，修改会自动应用到原 Buffer。

### 2. Grep-mode：外部工具的容器
当你搜的是“整个项目”或“磁盘文件”时，Emacs 会调用外部工具（如 `ripgrep`）。`Grep-mode` 只是一个展示外部搜索结果的窗口。因为它指向的是磁盘上的物理文件，所以需要 `wgrep` 这种专门的“补丁”来解析路径并实施文件修改。
*   **如何编辑：** 使用 `wgrep` 提供的快捷键（通常是 `C-c C-p`）。
*   **如何保存：** 修改完后按 **`C-c C-c`**，`wgrep` 会负责把改动写进硬盘。

---

## 三、 为什么 ripgrep 是现代搜索的首选？

在 `grep-mode` 的家族中，`consult-ripgrep` 是目前最推荐的工具。它调用的 `ripgrep (rg)` 之所以快到飞起，不仅仅是因为它用 **Rust** 编写，更因为以下几点硬核优化：

1.  **SIMD 加速**：利用 CPU 的硬件级指令（一次处理多组数据）扫描字符串，像排雷探测器一样瞬间扫过。
2.  **无 GC 开销**：Rust 没有垃圾回收机制，不会在搜索中途“喘气”（停顿）。
3.  **天生并行**：充分利用多核 CPU，多线程同时检索不同目录。
4.  **聪明地偷懒**：原生支持 `.gitignore`。它知道跳过 `node_modules` 和二进制文件，不搜索“垃圾”数据，速度自然快。

---

## 四、 实战技巧：如何根据需求选择？

### 场景 A：我只想修改当前文件中的几十处匹配
**推荐流程：**
1. `M-x consult-line` 搜索。
2. `embark-export` 导出 Buffer。
3. **按 `e` 进入 `occur-edit-mode`**（注意：Evil 用户可能需要先按 `C-z` 切换模式）。
4. 修改完毕，`C-c C-c` 提交。

### 场景 B：我想修改整个项目或特定目录
**推荐流程：**
1. `M-x consult-ripgrep`。
   - *小技巧*：如果只想搜当前目录，可以先按 **`C-u`** 再运行该命令，手动指定目录。
2. `embark-export` 导出 Buffer。
3. **执行 `wgrep` 流程**（`C-c C-p`）。
4. 修改完毕，`C-c C-c` 保存到磁盘。

---

## 五、 总结

在 Emacs 中，工具的选择往往取决于 **“数据在哪里”**。
*   数据在**内存**里 $\rightarrow$ 用 **Occur** $\rightarrow$ 按 **`e`** 编辑。
*   数据在**硬盘**里 $\rightarrow$ 用 **Grep/Ripgrep** $\rightarrow$ 用 **`wgrep`** 编辑。

搞清楚了这一点，你就能完美解决“Buffer 只读”的困扰，真正实现“搜索即编辑”的高效工作流。

