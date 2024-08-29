---
date: 2024-08-29T18:20:07+08:00
title: 'GDB使用教程'
description: "GDB的基本命令和高级调试技巧"
featured_image: "/images/gdb.png"
images: ["/images/gdb.png"]
tags: ["tutorial", "GDB"]
categories: "Tutorials"
---
# GDB 基本命令

GDB 提供了丰富的命令来调试程序。以下是一些常用的基本命令及其详细用法。
## 1. 启动 GDB

**命令**：`gdb [options] [program]`

- **直接启动并加载程序**：
 ```bash
  gdb ./your_program
  ```
  这会加载可执行文件 `your_program`，但不会立即运行程序。
- **加载程序并附加核心转储文件**：
  ```bash
  gdb ./your_program core
  ```
 通过加载核心转储文件，可以查看程序崩溃时的状态。
- **附加到正在运行的进程**：
 ```bash
  gdb -p <pid>
  ```
  通过进程 ID（PID）附加到正在运行的进程，这在需要调试后台进程时很有用。
## 2. 运行程序

**命令**：`run [args]`

- **启动并运行程序**：
```bash
  run
   ```
  如果程序需要命令行参数，可以在 `run` 后面加上参数：
  ```bash
  run arg1 arg2
  ```
- **重新运行程序**：如果程序在调试过程中已经运行过一次，可以使用 `run` 重新启动。GDB 会提示是否重新启动，输入 `y` 即可。
## 3. 设置断点

**命令**：`break [location] [if condition]`

- **在函数入口设置断点**：
  ```bash
  break main
  ```
- **在指定行号设置断点**：
  ```bash
  break 42
  ```
- **在特定文件的行号设置断点**：
  ```bash
  break example.c:10
  ```
- **条件断点**：仅在满足特定条件时触发。
  ```bash
  break 30 if x == 5
  ```
- **查看所有断点**：
  ```bash
  info breakpoints
  ```
- **删除断点**：
  ```bash
  delete <breakpoint-number>
  ```
## 4. 查看变量

**命令**：`print [expression]` 或 `p [expression]`

- **查看变量的当前值**：
  ```bash
  print variable_name
  ```
- **查看复杂表达式的结果**：
  ```bash
  print x + y * z
  ```
- **查看指针指向的内容**：
  ```bash
  print *ptr
  ```
- **以特定格式查看变量**（如十六进制、字符等）：
  ```bash
  print/x variable_name  # 以十六进制显示
  print/c variable_name  # 以字符显示
  ```
- **持续观察变量的变化**：
  ```bash
  display variable_name
  ```
  每次程序停止时，GDB 都会自动打印该变量的当前值。
## 5. 单步调试

**命令**：`next (n)` 和 `step (s)`

- **`next`**：执行下一行代码，不进入函数内部。
  ```bash
  next
  ``` 
  或者简写为：
  ```bash
  n
  ```
- **`step`**：执行下一行代码，如果是函数调用，会进入函数内部。
  ```bash
  step
  ```
  或者简写为：
  ```bash
  s
  ```
- **`finish`**：运行到当前函数的末尾并返回调用者。
  ```bash
  finish
  ```
## 6. 继续运行

**命令**：`continue (c)`

- **继续执行程序**直到下一个断点或程序结束：
  ```bash
  continue
  ```
  或者简写为：
  ```bash
  c
  ```
## 7. 查看调用栈

**命令**：`backtrace (bt)`

- **查看当前调用栈**：显示当前函数调用的层级关系，帮助分析调用路径。
  ```bash
  backtrace
  ```
  或者简写为：
  ```bash
  bt
  ```
- **查看调用栈的特定范围**：
  ```bash
  bt 3  # 只显示栈顶的 3 层调用
  ```
- **查看栈帧中的变量**：
  ```bash
  frame 1  # 切换到第 1 层栈帧
  info locals  # 显示当前栈帧中的局部变量
  ```
## 8. 查看源代码

**命令**：`list (l)`

- **查看当前执行点附近的源代码**：
  ```bash
  list
  ```
  或者简写为：
  ```bash
  l
  ```
- **查看特定行附近的代码**：
  ```bash
  list 10
  ```
  这将显示第 10 行附近的代码。
- **查看特定函数的代码**：
  ```bash
  list factorial
  ```
# GDB 高级调试技巧

## 1. 条件断点

**命令**：`break [location] if [condition]`

条件断点允许程序在特定条件满足时才暂停执行。例如，当你想在变量达到某个值时才触发断点，可以使用条件断点：

```bash
break 42 if x == 5  # 仅当 x 等于 5 时，在第 42 行触发断点
```
这种方式在调试循环或反复出现的函数调用时特别有用。

## 2. 观察点（Watchpoints）

**命令**：`watch [expression]`

观察点用于监视变量或内存位置的值变化。每当表达式的值发生变化时，程序会暂停执行。这在调试数据相关问题时非常有用：

```bash
watch variable_name  # 当 variable_name 发生变化时暂停执行
```
还可以设置条件观察点，只有在特定条件下才监视变量变化：
```bash
watch variable_name if variable_name > 100
```
## 3. 修改变量值

**命令**：`set variable [name] = [value]`

GDB 允许在调试过程中直接修改变量的值，这样可以测试不同场景下的程序行为，而无需重新编译：
```bash
set variable x = 10  # 将变量 x 的值设置为 10
```
这种方法在需要测试边界条件或错误修复时非常有帮助。

## 4. 调试共享库

**命令**：`info sharedlibrary`

在调试使用共享库的程序时，了解哪些库已加载非常重要。可以使用以下命令查看加载的共享库：
```bash
info sharedlibrary
```
要在共享库的加载时自动断点，可以使用：
```bash
set stop-on-solib-events 1
```
这将使 GDB 在加载每个共享库时暂停。

## 5. 调试多线程程序

**命令**：`info threads`, `thread [thread-id]`

GDB 支持多线程调试，可以查看和切换不同的线程：

- **查看所有线程**：
```bash
info threads
```
- **切换到特定线程**：
```bash
thread 2  # 切换到线程 2
```
- **设置线程断点**：
```bash
break 42 thread 2  # 仅在线程 2 的第 42 行触发断点
```
## 6. 追踪函数调用（Catchpoints）

**命令**：`catch [event]`

Catchpoints 用于捕捉特定的程序事件，比如异常、信号、或函数调用。常见的用法包括捕获 C++ 异常或程序收到的信号：

- **捕获 C++ 异常**：
```bash
catch throw  # 捕获所有 C++ 异常抛出事件
```
- **捕获函数调用**：
```bash
catch syscall open  # 捕获所有调用 open 系统调用的事件
```
## 7. 使用 TUI 模式

**命令**：`tui enable`

GDB 提供 TUI（Text User Interface）模式，通过窗口方式同时显示源代码、寄存器和变量等信息。进入 TUI 模式：
```bash
tui enable  # 启用 TUI 模式
```
可以在 TUI 模式下使用 `Ctrl+X`、`Ctrl+A` 切换窗口，方便查看调试信息。

## 8. 记录和回放（Replay）

**命令**：`record`

GDB 支持记录程序执行过程并回放，用于检查程序状态的变化。要开始记录：
```bash
record
```
## 9. 宏替换

**命令**：`define [macro-name]`

GDB 支持创建自定义命令，通过宏定义来简化复杂的调试过程。例如：
```bash
define myprint
print variable_name
end
```
现在可以使用 `myprint` 命令来查看 `variable_name` 的值。

## 10. 自定义 GDB 初始化文件

可以在 `~/.gdbinit` 文件中编写自定义命令和配置，GDB 启动时会自动加载。例如，预先设置常用的断点或变量观察点。
```bash
# 在 ~/.gdbinit 文件中
break main
set pagination off
```
