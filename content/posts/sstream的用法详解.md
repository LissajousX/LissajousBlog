---
date: 2025-07-03T17:23:31+08:00
title: sstream的用法详解
description: sstream的用法详解
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - cpp11
categories: Tutorials
---

# C++ 中的 `<sstream>` 全解析：istringstream、ostringstream 与 stringstream 的用法详解

C++ 提供的 `<sstream>` 头文件中包含了三个非常有用的类：`istringstream`、`ostringstream` 和 `stringstream`。它们提供了类似 `cin` / `cout` 的接口，专门用于**字符串的流式读写**，是处理字符串与各种类型之间转换的强大工具。

---

## 📌 基本介绍

|类名|用途|读写能力|类似于|
|---|---|---|---|
|`istringstream`|从字符串中读取内容|输入流（只读）|`std::cin`|
|`ostringstream`|向字符串中写入内容|输出流（只写）|`std::cout`|
|`stringstream`|读写字符串内容|输入 + 输出|`iostream`|

---

## 📘 1. `std::istringstream` —— 从字符串中提取数据

### 常用 API

|方法|描述|
|---|---|
|`istringstream(str)`|用字符串初始化流|
|`iss.str()`|获取当前的字符串内容|
|`iss.str("new str")`|重新设置流的字符串|
|`iss >> var`|从流中提取变量（空格分隔）|
|`std::getline(iss, s, delim)`|读取直到分隔符（默认换行）|

### 示例：将字符串中的数字提取为整数

```cpp
#include <sstream>
#include <iostream>

void parse_numbers(const std::string& input) {
    std::istringstream iss(input);
    int x;
    while (iss >> x) {
        std::cout << "Read: " << x << "\n";
    }
}

// 调用：parse_numbers("10 20 30");
```

---

## 📗 2. `std::ostringstream` —— 构造字符串输出

### 常用 API

|方法|描述|
|---|---|
|`ostringstream()`|创建空流|
|`oss.str()`|获取流中生成的字符串|
|`oss.str("...")`|重新设置流中的字符串（无效化之前内容）|
|`oss << data`|向流中写入数据|
|`oss.clear()`|清除错误状态（用于重复使用）|

### 示例：构建调试字符串

```cpp
#include <sstream>
#include <string>
#include <iostream>

std::string format_person(const std::string& name, int age) {
    std::ostringstream oss;
    oss << "Name: " << name << ", Age: " << age;
    return oss.str();
}
```

---

## 📙 3. `std::stringstream` —— 可读可写的字符串流

### 常用 API（结合 `istringstream` 和 `ostringstream`）

|方法|描述|
|---|---|
|`stringstream()`|创建空流|
|`ss.str()`|获取或设置字符串|
|`ss << val`|写入数据|
|`ss >> val`|读取数据|
|`std::getline(ss, s, delim)`|分隔符读取|
|`ss.clear()`|清除状态位（用于重用流）|

### 示例：构造字符串并立即解析内容

```cpp
#include <sstream>
#include <string>
#include <iostream>

void demo_stringstream() {
    std::stringstream ss;
    ss << "42 hello 3.14";

    int i;
    std::string word;
    double d;

    ss >> i >> word >> d;
    std::cout << i << " " << word << " " << d << "\n";
}
```

---

## 🎯 高级应用：自定义分隔符解析 CSV

```cpp
#include <sstream>
#include <vector>
#include <string>
#include <iostream>

std::vector<std::string> split_csv(const std::string& line) {
    std::vector<std::string> result;
    std::stringstream ss(line);
    std::string token;
    while (std::getline(ss, token, ',')) {
        result.push_back(token);
    }
    return result;
}
```

---

## ❗ 注意事项与技巧

|小技巧 / 陷阱|说明|
|---|---|
|流状态清空需要调用 `clear()`|否则可能导致下一次读写失败|
|流内容重置可用 `str("")`|清除原内容|
|字符串中的多个空格默认被忽略|提取时以空白符分隔|
|可以重载 `<<` / `>>` 运算符实现自定义类型的流操作|非常适合序列化与反序列化|

---

## 🔚 总结

`sstream` 是一个在 C++ 开发中非常实用却常被忽视的工具，它弥补了 C++ 在字符串处理上的一些短板，尤其适合：

- 字符串与数字之间的转换
    
- 字符串格式化输出
    
- 字符串分割 / 解析
    
- 文本协议 / 日志格式处理
    

掌握 `stringstream`，可以让你写出更加优雅、简洁、可维护的字符串处理代码。
