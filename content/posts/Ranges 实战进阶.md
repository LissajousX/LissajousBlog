---
date: 2025-07-08T18:56:19+08:00
title: Ranges 实战进阶
description: Ranges 实战进阶
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - mcpp
categories: Tutorials
---

# 🚀 Ranges 实战进阶：滑动窗口、zip_view、协程结合与性能分析

---

## 🧊 1. 实战：用 `ranges` 实现滑动窗口最大值（Monotonic Queue）

滑动窗口问题是算法题常客，比如 LeetCode 239：

> 给定数组 nums 和窗口大小 k，返回每个窗口内的最大值。

我们可以使用 `ranges` + 自定义 `view` 实现一个惰性滑动窗口视图。

### ✨ 目标接口：

```cpp
auto r = sliding_window(v, 3);
```

### 🧩 自定义 sliding_window view：

```cpp
template <std::ranges::view V>
requires std::ranges::forward_range<V>
class sliding_window_view : public std::ranges::view_interface<sliding_window_view<V>> {
    V base_;
    size_t k_;

public:
    sliding_window_view(V base, size_t k) : base_(std::move(base)), k_(k) {}

    struct iterator {
        std::ranges::iterator_t<V> first, last, end;
        size_t k;

        bool operator!=(const iterator& other) const {
            return first != other.first;
        }

        auto operator*() const {
            return std::ranges::subrange(first, last);
        }

        iterator& operator++() {
            ++first;
            ++last;
            return *this;
        }
    };

    auto begin() {
        auto first = std::ranges::begin(base_);
        auto last = first;
        std::ranges::advance(last, std::min(k_, std::ranges::distance(first, std::ranges::end(base_))));
        return iterator{first, last, std::ranges::end(base_), k_};
    }

    auto end() {
        auto e = std::ranges::end(base_);
        return iterator{e, e, e, k_};
    }
};

template <std::ranges::viewable_range R>
auto sliding_window(R&& r, size_t k) {
    return sliding_window_view<std::ranges::views::all_t<R>>(std::views::all(std::forward<R>(r)), k);
}
```

### ✅ 用法：

```cpp
std::vector<int> v{1, 3, -1, -3, 5, 3, 6, 7};

for (auto w : sliding_window(v, 3)) {
    int max = *std::ranges::max_element(w);
    std::cout << max << " ";
}
// 输出: 3 3 5 5 6 7
```

---

## 🔗 2. 实现一个简版 `zip_view`

标准库暂未实现 `views::zip`，我们可以用结构绑定自定义一个。

### 💡 核心思想：

并行遍历两个或多个范围，组合成一个 tuple 或 pair。

### 🚧 简化 zip_view（仅支持两个 range）

```cpp
template <std::ranges::input_range R1, std::ranges::input_range R2>
class zip_view {
    R1 r1_;
    R2 r2_;

public:
    zip_view(R1 r1, R2 r2) : r1_(std::move(r1)), r2_(std::move(r2)) {}

    struct iterator {
        std::ranges::iterator_t<R1> it1;
        std::ranges::iterator_t<R2> it2;

        auto operator*() const {
            return std::pair(*it1, *it2);
        }

        iterator& operator++() {
            ++it1;
            ++it2;
            return *this;
        }

        bool operator!=(const iterator& other) const {
            return it1 != other.it1 && it2 != other.it2;
        }
    };

    iterator begin() {
        return {std::ranges::begin(r1_), std::ranges::begin(r2_)};
    }

    iterator end() {
        return {std::ranges::end(r1_), std::ranges::end(r2_)};
    }
};

template <std::ranges::viewable_range R1, std::ranges::viewable_range R2>
auto zip(R1&& r1, R2&& r2) {
    return zip_view<std::ranges::views::all_t<R1>, std::ranges::views::all_t<R2>>(
        std::views::all(std::forward<R1>(r1)), std::views::all(std::forward<R2>(r2)));
}
```

### ✅ 用法：

```cpp
std::vector<int> a{1, 2, 3};
std::vector<std::string> b{"one", "two", "three"};

for (auto [x, y] : zip(a, b)) {
    std::cout << x << ": " << y << "\n";
}
```

---

## 🌈 3. 与协程结合的展望（C++20）

C++20 的协程与 ranges 是天然搭档。例如你可以编写 `generator<T>`：

```cpp
generator<int> iota(int start, int end) {
    for (int i = start; i < end; ++i)
        co_yield i;
}
```

然后包装为一个 `view`，实现懒惰无限序列、异步生成等。

目前标准库并未集成 coroutine-ranges，但 `cppcoro`, `generator` 等库已经实现了类似功能，未来 C++26 将会加强这种支持。

---

## 📊 4. 性能与对比分析

|项目|传统 STL|ranges 写法|
|---|---|---|
|可读性|中|高（更像自然语言）|
|组合性|低|高（管道链式）|
|惰性计算|无|✅ 默认惰性|
|性能|稍快|与 STL 相近（懒惰 + inline）|
|Debug 类型|简单|❗可能复杂（需配合 `decltype(auto)`）|
|拓展能力|需手写|✅ 可组合 views、自定义 view|

> ✅ 推荐实践：在性能要求不极端的情况下，优先使用 `ranges`，写法更现代、语义更清晰。

---

## 📘 最终总结（完整版）

|分类|内容|
|---|---|
|ranges 算法|`ranges::sort`, `ranges::find`, `ranges::copy` 等|
|views|`filter`, `transform`, `take`, `drop`, `iota`, `reverse`|
|链式组合|`r|
|自定义 view|可扩展行为（如滑窗、分组等）|
|zip_view / sliding_window|实现复合逻辑|
|range concepts|用于模板约束或接口泛化|
|协程结合|可构建惰性数据流（未来趋势）|
|容器转换|`to<vector>()` 或 `vector(r.begin(), r.end())`|
|性能建议|惰性默认行为，适合大数据惰性处理场景|
