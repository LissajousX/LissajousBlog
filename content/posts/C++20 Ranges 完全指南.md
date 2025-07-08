---
date: 2025-07-08T18:54:07+08:00
title: C++20 Ranges 完全指南
description: C++20 Ranges 完全指南
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - mcpp
categories: Tutorials
---

# C++20 Ranges 完全指南：懒惰计算、链式操作与函数式编程范式

> 在本篇文章中，我们将深入介绍 C++20 引入的 ranges 库，包括 `std::ranges` 算法、`std::views` 视图、链式组合技巧、容器转换等高级技巧，助你掌握这一现代 C++ 中的函数式利器。

---

## 什么是 Ranges？

传统的 C++ STL 算法，如 `std::sort`、`std::copy` 等，通常需要传入 `begin()` 与 `end()`。而 `ranges` 则基于“**范围对象**”操作容器，具备以下优势：

- 更简洁：支持直接传容器。
    
- 可组合：支持管道操作 `|` 实现链式风格。
    
- 惰性计算：`views` 不立即生成结果，节省资源。
    
- 函数式编程风格：更接近 Python、Rust 等现代语言的风格。
    

---

## 基础使用：`ranges` 算法

C++20 在 `<algorithm>` 中添加了 ranges 版本的 STL 算法（位于 `std::ranges::` 命名空间中）：

```cpp
#include <vector>
#include <ranges>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> v{5, 1, 3, 2};

    std::ranges::sort(v);  // 排序
    std::ranges::for_each(v, [](int x){ std::cout << x << " "; });  // 输出：1 2 3 5
}
```

几乎所有常用算法都存在 ranges 版本：`sort`、`find`、`copy`、`for_each`、`unique`、`remove` 等。

---

## Views：懒惰计算视图

`std::views` 提供一系列可组合的“视图”操作，它们不会立即计算结果，而是在访问元素时才执行。

### 常见视图（`std::views::`）：

|视图|功能说明|
|---|---|
|`filter`|过滤元素|
|`transform`|元素映射（变换）|
|`take(n)`|取前 n 个元素|
|`drop(n)`|跳过前 n 个元素|
|`reverse`|反转序列|
|`iota(a, b)`|类似 Python 的 `range(a, b)`|

```cpp
std::vector<int> v{1, 2, 3, 4, 5, 6};

auto r = v 
    | std::views::filter([](int x){ return x % 2 == 0; })
    | std::views::transform([](int x){ return x * x; });

for (int x : r) std::cout << x << " ";  // 输出：4 16 36
```

---

## 链式组合与管道操作符

使用 `|` 管道符可以组合多个 `views` 操作，形成惰性计算的管道链，避免中间变量：

```cpp
auto r = vec
    | std::views::filter(...)
    | std::views::transform(...)
    | std::views::take(10);
```

这种方式不仅可读性强，还能高效处理大规模数据。

---

## 常见视图与范例

### 🎯 `views::filter`

```cpp
auto even = vec | std::views::filter([](int x){ return x % 2 == 0; });
```

### 🔁 `views::transform`

```cpp
auto square = vec | std::views::transform([](int x){ return x * x; });
```

### ➕ 链式组合示例

```cpp
auto result = vec
    | std::views::filter([](int x){ return x > 3; })
    | std::views::transform([](int x){ return x * 10; })
    | std::views::take(3);
```

### 🔢 `views::iota`

```cpp
for (int x : std::views::iota(1, 6)) {
    std::cout << x << " ";  // 1 2 3 4 5
}
```

---

## 结果转换为容器

因为 views 是惰性序列，若需要结果容器（如 `vector`），可以手动转换：

### ✅ C++20 写法

```cpp
std::vector<int> out(r.begin(), r.end());
```

### ✅ C++23 更方便：

```cpp
auto out = r | std::ranges::to<std::vector>();
```

> 若使用 C++20，可以自己写一个 `to_vector()` 工具函数进行转换。

---

## 完整案例演示

### 示例：提取偶数并平方，保存为 `std::vector`

```cpp
#include <vector>
#include <ranges>
#include <iostream>

int main() {
    std::vector<int> data{1,2,3,4,5,6,7,8};

    auto r = data
        | std::views::filter([](int x){ return x % 2 == 0; })
        | std::views::transform([](int x){ return x * x; });

    std::vector<int> result(r.begin(), r.end());  // 结果为 {4, 16, 36, 64}

    for (int x : result) std::cout << x << " ";
}
```

---

## 拓展库推荐

如果你想使用更多 views（如 `zip`, `enumerate`, `chunk`），推荐以下库：

- [range-v3](https://github.com/ericniebler/range-v3)：C++20 ranges 的鼻祖，支持更多视图操作。
    
- [cppcoro](https://github.com/lewissbaker/cppcoro)：提供协程风格的 ranges。
    

---

## 总结

|特性|说明|
|---|---|
|`ranges::` 算法|替代传统 STL 算法，支持容器直接传参|
|`views::` 视图|支持懒惰链式处理，性能更优|
|管道 `|` 操作|
|容器转换|可通过迭代器或 `ranges::to` 转换为 vector 等|
