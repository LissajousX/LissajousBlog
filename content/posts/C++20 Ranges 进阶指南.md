---
date: 2025-07-08T18:55:27+08:00
title: C++20 Ranges 进阶指南
description: C++20 Ranges 进阶指南
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - mcpp
categories: Tutorials
---

# 🔍 C++20 Ranges 进阶指南：Subrange、概念、扩展 Views 与自定义 View 实现

---

## 📦 1. `std::ranges::subrange`：显式表示一个范围

`std::ranges::subrange` 是一种轻量级结构，用于包装 `[begin, end)` 的一对迭代器，可以传给 ranges 算法。

### ✨ 用法

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v{10, 20, 30, 40, 50};

    auto first = v.begin() + 1;
    auto last = v.begin() + 4;

    std::ranges::subrange r(first, last);

    for (auto x : r)
        std::cout << x << " ";  // 输出: 20 30 40
}
```

### ✅ 优点

- 可以显式构造一个不完整容器的范围。
    
- 可作为 `views` 或算法的中间输出传递。
    
- 完全兼容 `std::ranges` 系统。
    

---

## 🧱 2. `range` Concepts：约束范围类型

C++20 引入了 Concepts，为 `ranges` 提供了精确的约束判断。

### 🌐 常用 Concepts

|Concept|意义|
|---|---|
|`std::ranges::range`|是一个 range（有 begin/end）|
|`std::ranges::input_range`|可输入遍历|
|`std::ranges::forward_range`|可多次遍历|
|`std::ranges::view`|是惰性视图|
|`std::ranges::sized_range`|支持 `size()`|
|`std::ranges::common_range`|`begin` 和 `end` 类型相同|

### ✅ 示例：限制函数参数为 forward_range

```cpp
template<std::ranges::forward_range R>
void print_range(const R& r) {
    for (auto&& x : r) std::cout << x << " ";
}
```

---

## 🧩 3. 自定义 View 实现（DIY）

如果你需要自定义一个特殊的视图逻辑，比如“滑动窗口”、“滑动最大值”等，可以自定义 view。

### 🛠 模板

```cpp
template<std::ranges::view V>
requires std::ranges::forward_range<V>
class my_view : public std::ranges::view_interface<my_view<V>> {
    V base_;

public:
    my_view() = default;
    my_view(V base) : base_(std::move(base)) {}

    auto begin() {
        return std::ranges::begin(base_);
    }

    auto end() {
        return std::ranges::end(base_);
    }
};
```

- 必须继承 `view_interface`（提供 `empty()`, `size()` 等接口）。
    
- 内部需保存 base view 或 range。
    
- 必须满足 `begin()`/`end()` 的要求。
    

这是构造自定义视图的基础框架，你可以在 `begin()` 和 `end()` 中返回定制迭代器，实现自定义行为。

---

## 🧪 4. range-v3 中的扩展 views

标准库中的 `views::zip`、`views::enumerate` 等尚未收录，但可用第三方库 [`range-v3`](https://github.com/ericniebler/range-v3)。

### 示例 1：`views::zip`

```cpp
#include <range/v3/view/zip.hpp>

std::vector<int> a{1, 2, 3};
std::vector<char> b{'a', 'b', 'c'};

auto zipped = ranges::views::zip(a, b);

for (auto [x, y] : zipped) {
    std::cout << x << " - " << y << "\n";  // 输出: 1 - a, 2 - b, ...
}
```

### 示例 2：`views::enumerate`

```cpp
#include <range/v3/view/enumerate.hpp>

std::vector<std::string> v{"apple", "banana", "pear"};

for (auto [i, s] : ranges::views::enumerate(v)) {
    std::cout << i << ": " << s << "\n";
}
```

> 如果你在 C++20 标准库中找不到这些 view，可以手动移植或使用 range-v3。

---

## ⚠️ 5. 注意事项与性能陷阱

- **惰性遍历，非缓存**：`views` 默认每次访问都重新计算，适合一次遍历，不适合频繁随机访问。
    
- **嵌套视图需注意生命周期**：不要返回本地 view，确保引用合法。
    
- **组合嵌套可能产生难以调试的类型**：可以使用 `decltype(auto)` 保留类型。
    

```cpp
auto r = vec | std::views::filter(...) | std::views::transform(...);
// 不建议写：std::vector<int> r = ... // 错误
```

- **转容器需手动 copy**：views 是懒惰的，转 `vector` 需显式调用 `to_vector()` 或构造。
    

---

## 🧠 总结升级版

|模块|内容|
|---|---|
|`ranges::subrange`|显式构造任意范围|
|Concepts|限制参数/模板范围类型|
|自定义 view|构造新视图逻辑（如滑窗、分组等）|
|range-v3 扩展|`zip`, `enumerate`, `join` 等非标视图|
|注意事项|views 非缓存、组合类型复杂、手动容器转换|
