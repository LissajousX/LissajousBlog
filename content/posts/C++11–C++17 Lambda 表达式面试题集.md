---
date: 2025-05-14T11:33:41+08:00
title: C++11–C++17 Lambda 表达式面试题集
description: C++11–C++17 Lambda 表达式面试题集
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - cpp11
categories: Tutorials
---


本文给出一套基于 **C++11 至 C++17** 的 Lambda 表达式面试题，分为基础理解、STL 应用、底层原理和语言障眼等组成。

---

## 🔹 一、基础理解题

### Q1. 以下代码的输出是什么？为什么？

```cpp
int a = 10;
auto lambda = [a]() mutable {
    a += 5;
    std::cout << a << std::endl;
};
lambda();
std::cout << a << std::endl;
```

**答案：**

```
15
10
```

---

### Q2. 判断以下语法是否合法？

```cpp
int x = 5;
auto f = [&x, x](int y) {
    return x + y;
};
```

**答案：**

- C++11: 不合法
    
- C++14+: 合法
    

---

### Q3. 写出一个保证 counter == 5 的 lambda

```cpp
int counter = 0;
std::function<void()> f;

{
    int temp = 5;
    // 在这里定义 lambda
    f = [val = temp, &counter]() {
        counter = val;
    };
}

f();  // counter == 5
```

---

## 🔹 二、STL 应用题

### Q4. 用 Lambda 对字符串数组按长度排序

```cpp
std::vector<std::string> vec = {"cat", "elephant", "dog", "antelope"};
std::sort(vec.begin(), vec.end(), [](const auto& a, const auto& b) {
    return a.length() < b.length();
});
```

---

### Q5. 过滤 vector，保留 >10 元素

```cpp
std::vector<int> nums = {3, 12, 7, 18, 5};
int threshold = 10;
std::vector<int> result;

std::copy_if(nums.begin(), nums.end(), std::back_inserter(result),
             [threshold](int x) { return x > threshold; });
```

---

## 🔹 三、底层原理题

### Q6. 编译器将下列 lambda 扩展成怎样的类？

```cpp
int a = 1, b = 2;
auto f = [=, &b](int x) {
    return a + b + x;
};
```

**类类似实现：**

```cpp
class Lambda {
    int a;
    int& b;
public:
    Lambda(int a_, int& b_) : a(a_), b(b_) {}
    int operator()(int x) const {
        return a + b + x;
    }
};
```

---

### Q7. 为什么无损损 lambda 可转换为函数指针？

```cpp
auto f = [](int x) { return x + 1; };
int (*fp)(int) = f;  // 正确

int a = 10;
auto g = [a](int x) { return a + x; };
// int (*gp)(int) = g;  // 错误
```

---

### Q8. 泛型 lambda 怎样实现？

```cpp
auto f = [](auto x, auto y) { return x + y; };
```

**等价类型编译器扩展：**

```cpp
struct Lambda {
    template<typename T1, typename T2>
    auto operator()(T1 x, T2 y) const {
        return x + y;
    }
};
```

---

## 🔹 四、障眼题 & 扩展

### Q9. 是否存在悬排引用？

```cpp
std::function<void()> f;
{
    int x = 42;
    f = [&x]() {
        std::cout << x << std::endl;
    };
}
f(); // 未定行为！
```

---

### Q10. 此 lambda 的返回类型是否合法？

```cpp
auto f = [](bool b) {
    if (b) return 1;
    else return 3.14;
};
```

**错误：返回类型不符合**

**修复：**

```cpp
auto f = [](bool b) -> double {
    return b ? 1 : 3.14;
};
```

---

## ✅ Bonus: 手写 lambda 行为类

### Q11. 手写等价类

```cpp
int a = 10, b = 20;
auto f = [a, &b](int x) { return a + b + x; };
```

**等价类：**

```cpp
class SimulatedLambda {
    int a;
    int& b;
public:
    SimulatedLambda(int a_, int& b_) : a(a_), b(b_) {}
    int operator()(int x) const {
        return a + b + x;
    }
};
```