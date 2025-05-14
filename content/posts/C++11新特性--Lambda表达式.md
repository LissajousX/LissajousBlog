---
date: 2025-05-14T11:00:59+08:00
title: C++11新特性--Lambda表达式
description: C++11新特性--Lambda表达式
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - cpp11
categories: Tutorials
---
# 基本介绍

C++11 引入了 **Lambda 表达式（Lambda Expression）**，它是一种用于定义 **匿名函数对象** 的语法。Lambda 表达式让我们可以更方便地编写内联函数，尤其是在需要临时函数行为的场景，比如 `std::sort()`、`std::for_each()`、回调函数等。

## 一、基本语法

```cpp
[capture](parameters) -> return_type {
    function_body
};
```

各部分含义如下：

| 部分 | 含义 |
| ---- | ---- |
| [capture] | 捕获外部变量的方式（值捕获、引用捕获等） |
| (params) | 参数列表（类似普通函数） |
| -> type | 返回类型（可省略，编译器会推导） |
| { body } | 函数体 |

## 二、基本示例


```cpp
#include 
int main() {
    auto add = [](int a, int b) {
        return a + b;
    };
    std::cout << add(2, 3) << std::endl;  // 输出 5
}
```


## 三、捕获变量（capture）

捕获列表 `[ ]` 指定了 Lambda 可以使用的外部变量，有以下方式：

### 1. 值捕获（by value）

```cpp
int x = 10;
auto f = [x]() {
    std::cout << x << std::endl;  // 拷贝 x
};
```

### 2. 引用捕获（by reference）

```cpp
int x = 10;
auto f = [&x]() {
    x += 1;
};
```

### 3. 混合捕获

```cpp
int a = 1, b = 2;
auto f = [a, &b]() {
    // a 是值捕获，b 是引用捕获
};
```

### 4. 隐式捕获

- `[=]`：所有使用的外部变量按 **值** 捕获
- `[&]`：所有使用的外部变量按 **引用** 捕获

## 四、返回类型

如果返回值复杂或不易推导，可以使用 `-> return_type` 指定：

```cpp
auto f = [](int a, int b) -> double {
    return a / (double)b;
};
```

## 五、可变 Lambda（mutable）

默认情况下，值捕获的变量在 Lambda 内部是 `const` 的。如果想修改这些值，可以使用 `mutable`：

```cpp
int x = 10;
auto f = [x]() mutable {
    x += 5;
    std::cout << x << std::endl;  // 输出 15
};
f();
std::cout << x << std::endl;  // 输出 10，外部不变
```


## 六、Lambda 与标准算法结合


```cpp
#include 
#include 

std::vector v = {1, 2, 3, 4, 5};
std::for_each(v.begin(), v.end(), [](int x) {
    std::cout << x << " ";
});
```



## 七、Lambda 的类型和 std::function


Lambda 是编译器生成的匿名类，可以赋值给 `std::function`：


```cpp
#include 

std::function f = [](int a, int b) {
    return a + b;
};
```



## 总结：Lambda 的强大之处


- **内联定义函数逻辑**，避免冗余函数定义
- **方便与 STL 配合**
- 支持 **值/引用捕获**，灵活性强
- 实现简单的 **回调函数** 或事件处理逻辑

如果你有具体场景或者想结合 STL 算法深入了解，我可以给你举几个实用的例子。是否需要？

# 实际例子

## 一、结合标准算法的实用示例

### 示例1：std::sort 自定义排序


```cpp
#include 
#include 
#include 

int main() {
    std::vector v = {5, 1, 3, 4, 2};

    // 按降序排序
    std::sort(v.begin(), v.end(), [](int a, int b) {
        return a &gt; b;
    });

    for (int x : v) std::cout << x << " ";  // 输出：5 4 3 2 1
}
```

### 示例2：std::find_if 查找第一个满足条件的元素


```cpp
#include 
#include 
#include 

int main() {
    std::vector v = {1, 3, 5, 8, 10};

    auto it = std::find_if(v.begin(), v.end(), [](int x) {
        return x % 2 == 0;
    });

    if (it != v.end()) std::cout << "找到偶数：" << *it << std::endl;  // 输出 8
}
```

### 示例3：std::count_if 统计满足条件的个数


```cpp
#include 
#include 
#include 

int main() {
    std::vector v = {1, 4, 6, 9, 10};

    int count = std::count_if(v.begin(), v.end(), [](int x) {
        return x &gt; 5;
    });

    std::cout << "大于5的元素个数：" << count << std::endl;  // 输出 3
}
```

### 示例4：传递状态的 Lambda（通过捕获变量）


```cpp
#include 
#include 
#include 

int main() {
    std::vector v = {1, 2, 3, 4, 5};
    int threshold = 3;

    std::vector filtered;
    std::copy_if(v.begin(), v.end(), std::back_inserter(filtered), [threshold](int x) {
        return x &gt; threshold;
    });

    for (int x : filtered) std::cout << x << " ";  // 输出 4 5
}
```

## 二、Lambda 表达式的底层本质（编译器实现方式）


Lambda 是一种 **匿名函数对象**。本质上，编译器会为每个 Lambda 生成一个匿名的类并重载 `operator()`，例如：

```cpp
auto f = [](int x) { return x + 1; };
```

等价于：

```cpp
struct Lambda {
    int operator()(int x) const {
        return x + 1;
    }
};

Lambda f;
```

如果 Lambda 捕获了变量，编译器会将这些变量存储为类的成员变量。

## 三、lambda 与 std::function 的比较

- `auto f = [](int x) { return x + 1; };` —— 这时候 `f` 的类型是一个编译器生成的类，不能直接写出类型名。
- 如果你需要把 lambda 存入容器或传递给函数，你通常用 `std::function`：

```cpp
#include 

std::function func = [](int x) { return x + 1; };
```

注意：`std::function` 是有开销的（类型擦除、堆分配等），如果能使用 `auto`，通常性能更好。

## 四、可递归 Lambda

C++11 不支持 lambda 自引用自己的名字（因为没有名字），不过我们可以使用 `std::function` 来递归：

```cpp
#include 
#include 

int main() {
    std::function fib = [&](int n) -> int {
        if (n <= 1) return n;
        return fib(n - 1) + fib(n - 2);
    };

    std::cout << fib(10) << std::endl;  // 输出 55
}
```

## 五、总结要点

| 特性 | 描述 |
| ---- | ---- |
| [] 捕获列表 | 捕获外部变量（值/引用） |
| mutable | 修改捕获的值变量副本 |
| -> return_type | 明确返回类型 |
| 可与 STL 配合 | sort、find_if、count_if 等更清晰 |
| 类型为匿名类 | 实现 operator() 的闭包 |
| 与 std::function 配合 | 支持递归、类型擦除（但略有性能开销） |

#  C++14/C++17 中 Lambda 的扩展

C++14 和 C++17 对 Lambda 表达式进行了非常实用的增强，尤其是 **泛型 Lambda** 和 **更灵活的捕获方式**。

## ✅ 一、C++14：Lambda 表达式的增强

### 1. 泛型 Lambda（Generic Lambda）

C++14 允许你在 Lambda 的参数列表中使用 `auto`，表示参数类型自动推导 —— 这就使得 Lambda 本身变成模板函数的感觉。
#### ✅ 示例：

```cpp
auto add = [](auto a, auto b) {
    return a + b;
};

std::cout << add(1, 2) << std::endl;         // 输出 3（int）
std::cout << add(1.5, 2.5) << std::endl;     // 输出 4.0（double）
std::cout << add(std::string("a"), "b") << std::endl; // 输出 ab
```
### ✅ 背后原理：

泛型 Lambda 编译器实现其实就是自动为每个调用生成一个模板的 `operator()`。

### 2. 初始化捕获（Init-capture）

C++14 引入了一种 **按值初始化局部变量** 并捕获的方式，语法是：

```cpp
[variable_name = expression]
```
#### ✅ 示例：

```cpp
int x = 10;

auto f = [y = x + 5]() {
    std::cout << y << std::endl;  // 输出 15
};
f();
```

这种捕获方式非常适合将复杂表达式的结果捕获进 Lambda，**而不是捕获变量本身再在 Lambda 中求值**。


## ✅ 二、C++17：Lambda 表达式的增强

### 1. 默认捕获与显式混合捕获

在 C++11 中，不能同时使用 `[=]` 和 `&x`；而在 **C++17 中，允许组合默认捕获和显式捕获**：
#### ✅ 示例：

```cpp
int a = 1, b = 2;

auto f = [=, &b]() {
    // a 被值捕获，b 被引用捕获
    std::cout << a << " " << ++b << std::endl;
};
f();  // 输出：1 3（b 修改了）
```

同样也可以反过来 `[&, x]` 表示默认引用捕获，但 x 值捕获。

### 2. 捕获 this 指针的成员变量（C++17）

C++11 只能捕获 `this` 指针，然后在 lambda 中访问 `this->member`。
C++17 支持 **值捕获成员变量** —— 如果用 `[=]` 捕获，就相当于按值拷贝 `this` 所指对象的成员：

```cpp
struct MyClass {
    int x = 42;

    void print() {
        auto f = [=]() {
            std::cout << x << std::endl;  // 相当于 this->x
        };
        f();
    }
};
```

注意：实际上 `[=]` 捕获的是 `this`，只是 C++17 在语义上简化了访问方式。

### 3. Lambda 可以是 constexpr

C++17 开始允许 Lambda 成为 `constexpr`，即可以在编译期调用。
#### ✅ 示例：

```cpp
constexpr auto square = [](int x) {
    return x * x;
};

constexpr int y = square(5);  // 编译期计算 y = 25
```

## ✅ 总结


| 特性 | C++14 / C++17 状态 | 示例 |
| ---- | ---- | ---- |
| 泛型 Lambda | ✅ C++14 | [](auto a, auto b){...} |
| 初始化捕获 | ✅ C++14 | [val = expr] |
| 混合捕获（默认+显式） | ✅ C++17 | [=, &x] 或 [&, x] |
| 简化 this 捕获 | ✅ C++17 | [=] 可直接访问成员变量 |
| constexpr Lambda | ✅ C++17 | constexpr auto f = ...; |

# 底层编译器如何生成 Lambda 的类结构


## ✅ Lambda 的本质

Lambda 表达式在编译器眼中是一个 **匿名类（closure class）** 的实例，它 **重载了 operator()**。如果你写了一个 Lambda，编译器会自动帮你生成一个类似下面结构的类。

## 🔧 示例 1：无捕获 Lambda

```cpp
auto f = [](int x) { return x + 1; };
```

等价于（伪代码）：

```cpp
struct __Lambda_1 {
    int operator()(int x) const {
        return x + 1;
    }
};

__Lambda_1 f;
```


无捕获的 Lambda 可以 **隐式转换为函数指针**：

```cpp
int (*fp)(int) = [](int x) { return x + 1; };
```


## 🔧 示例 2：有捕获变量的 Lambda


```cpp
int a = 10;
auto f = [a](int x) { return x + a; };
```

编译器等价于：

```cpp
struct __Lambda_2 {
    int a;  // 捕获的变量成为成员

    __Lambda_2(int _a) : a(_a) {}

    int operator()(int x) const {
        return x + a;
    }
};

__Lambda_2 f(a);
```

✅ 注意：
- 捕获的变量作为类成员存储；
- `operator()` 默认为 `const` 成员函数；
- `mutable` 会移除这个 `const` 修饰。

## 🔧 示例 3：引用捕获

```cpp
int a = 10;
auto f = [&a](int x) { a += x; };
```

编译器等价于：

```cpp
struct __Lambda_3 {
    int& a;

    __Lambda_3(int& _a) : a(_a) {}

    void operator()(int x) const {
        a += x;
    }
};

__Lambda_3 f(a);
```

## 🔧 示例 4：mutable Lambda（值捕获但允许修改副本）


```cpp
int a = 10;
auto f = [a]() mutable { a += 1; std::cout << a; };
```

编译器生成的伪代码：

```cpp
struct __Lambda_4 {
    int a;

    __Lambda_4(int _a) : a(_a) {}

    void operator()() {
        a += 1;
        std::cout << a;
    }
};
```

注意 `operator()` 不再是 `const`，因为需要修改成员 `a`。


## ✅ 实际观察 Lambda 类：使用 clang++ -Xclang -ast-dump 或 MSVC /d1reportAllClassLayout

在实际项目中你可以用 `clang` 或 MSVC 工具查看 Lambda 对象的实际类结构。比如：


```sh
clang++ -std=c++14 -Xclang -ast-dump -fsyntax-only lambda.cpp
```


可以看到 Lambda 转换成了具体类结构，编译器自动生成了：

- 数据成员（捕获变量）
- `operator()`
- 构造函数（初始化捕获成员）
- `operator==`（C++20 起支持比较）
- 可能的 `constexpr` 或 `noexcept` 属性


## ✅ 特殊情况：泛型 Lambda（C++14+）


```cpp
auto f = [](auto x) { return x * x; };
```

等价于一个 **类模板的 operator()**：

```cpp
struct __Lambda {
    template
    auto operator()(T x) const {
        return x * x;
    }
};
```

这是 C++14 的一大增强：Lambda 自带模板机制。

## ✅ 总结

| 特性 | 底层实现 |
| ---- | ---- |
| 无捕获 Lambda | 类中无成员，支持函数指针转换 |
| 值捕获 | 捕获变量是类成员 |
| 引用捕获 | 成员是引用类型 |
| mutable | operator() 去掉 const |
| 泛型 Lambda | operator() 是模板函数 |
 **Lambda 本质就是带状态的函数对象**。
 
# 手动模拟一个 Lambda 的底层结构

**亲手实现一个类**，功能和下面的 Lambda 完全等价：

```cpp
int a = 10;
int b = 20;
auto lambda = [a, &b](int x) {
    return a + b + x;
};
```

## 🎯 目标：自己实现等价的类结构

### Lambda 等价于这样的调用方式：

```cpp
int result = lambda(5);  // 返回 a + b + x = 10 + 20 + 5 = 35
```

## 🛠 手动实现

我们来一步步写出它对应的类结构：

```cpp
#include 

struct MyLambda {
    int a;    // 值捕获的 a
    int& b;   // 引用捕获的 b

    // 构造函数：模拟捕获
    MyLambda(int a_, int& b_) : a(a_), b(b_) {}

    // operator() 模拟 lambda 的调用
    int operator()(int x) const {
        return a + b + x;
    }
};

int main() {
    int a = 10;
    int b = 20;

    MyLambda lambda(a, b);  // 模拟捕获
    std::cout << lambda(5) << std::endl;  // 输出 35
}
```

## ✅ 分析

| 部分 | 实现逻辑 |
| ---- | ---- |
| int a | 模拟值捕获 |
| int& b | 模拟引用捕获 |
| 构造函数 | 初始化捕获变量 |
| operator() | 模拟函数调用语法 lambda(x) |
| const 修饰 | 表示函数不会修改 a 和 b 的值 |

## 🔄 加上 mutable 的变体

Lambda：

```cpp
auto lambda = [a]() mutable {
    a += 1;
    std::cout << a;
};
```

等价类：

```cpp
struct MutableLambda {
    int a;

    MutableLambda(int a_) : a(a_) {}

    void operator()() {
        a += 1;
        std::cout << a << std::endl;
    }
};
```

注意：**没有 const**，因为 mutable 表示我们允许修改副本。



## 🧠 思考练习（你可以尝试实现）：


1. 模拟捕获 `[=]` 和 `[&]` 混合的更复杂结构；
2. 模拟一个泛型 Lambda，比如 `[](auto x, auto y) { return x + y; }`；
3. 给你实现的类加上 `constexpr` 或 `noexcept` 支持；
4. 给 `MyLambda` 添加 `clone()`、`equals()` 等高级能力，像标准函数对象那样可复制可比较。


## ✅ 总结

手动实现 Lambda 类的过程，你其实在做的是：
- 模拟 **闭包（Closure）结构**；
- 理解 **值捕获 vs 引用捕获的语义区别**；
- 实践 **函数对象（functor）模式**；
- 学会在模板、函数指针、`std::function` 等之间切换。

# C++ Lambda 面试题集

下面是一个系统化的 **C++ Lambda 面试题集（含解析 & 难度分级）**，分为 4 个阶段：**基础语法** → **STL 应用** → **底层原理** → **高级陷阱 & 模拟实现**。

## 🧩 第一部分：基础语法与行为（难度：★☆☆）

### ✅ **题 1：值捕获 vs 引用捕获**

```cpp
int x = 10;
auto f = [x]() mutable { x += 5; std::cout << x; };
f();
std::cout << x;
```

- 输出是什么？
- 为什么 `x` 没有被修改？

✔️ 答案：输出 `15 10`。`[x]` 是值捕获，`mutable` 允许修改的是副本。

### ✅ **题 2：Lambda 返回类型推导**

```cpp
auto f = [](bool b) {
    if (b) return 1;
    else return 3.14;
};
```

- 这段代码是否合法？
- 如果不合法，怎么修复？

❌ 非法：两个 `return` 的类型不同。
✔️ 修复方法一：手动指定返回类型：

```cpp
auto f = [](bool b) -> double {
    if (b) return 1;
    else return 3.14;
};
```

### ✅ **题 3：初始化捕获（C++14）**

```cpp
int x = 10;
auto f = [y = x + 5]() { std::cout << y; };
f();
```

- 输出什么？
- 初始化捕获在 C++11 中合法吗？

✔️ 输出 `15`；初始化捕获是 **C++14 引入的新特性**。


## 🧩 第二部分：Lambda 与 STL 应用（难度：★★☆）

### ✅ **题 4：std::sort + Lambda 排序**

按字符串长度从短到长排序：

```cpp
std::vector vec = {"apple", "kiwi", "banana"};
```

请写出排序语句。

```cpp
std::sort(vec.begin(), vec.end(), [](const std::string& a, const std::string& b) {
    return a.size() < b.size();
});
```

### ✅ **题 5：std::copy_if + 外部变量捕获**

```cpp
std::vector v = {3, 12, 5, 8, 20};
int threshold = 10;
std::vector filtered;

std::copy_if(v.begin(), v.end(), std::back_inserter(filtered), [threshold](int x) {
    return x &gt; threshold;
});
```

- 捕获 `threshold` 是否可以用 `[=]`？
- 如果使用 `[&]` 有何风险？

✔️ `[=]` 是值捕获，可安全使用；
⚠️ `[&]` 如果 threshold 是局部变量，会导致悬空引用风险。

## 🧩 第三部分：底层原理理解（难度：★★★）

### ✅ **题 6：解释这个 Lambda 是如何被编译器转换的？**

```cpp
int a = 10, b = 20;
auto f = [=, &b](int x) { return a + b + x; };
```

要求写出其等价的结构体类。

```cpp
struct Lambda {
    int a;
    int& b;

    Lambda(int a_, int& b_) : a(a_), b(b_) {}

    int operator()(int x) const {
        return a + b + x;
    }
};

Lambda f(a, b);
```

### ✅ **题 7：为什么无捕获 Lambda 能转换为函数指针？**

```cpp
auto f = [](int x) { return x + 1; };
int (*fp)(int) = f;
```

- 如果改成 `[x]`, 能转换吗？
- 为什么？


✔️ 原因：无捕获 Lambda 编译为 `static` 函数，**无状态**，所以可以退化为函数指针。
❌ 有捕获 Lambda 是一个 **有状态类对象**，不能退化。

### ✅ **题 8：泛型 Lambda 的 operator() 是如何工作的？**

```cpp
auto f = [](auto x) { return x * x; };
```

- 编译器为此生成了什么？
- 这可以理解为什么结构？

✔️ 类似：

```cpp
struct Lambda {
    template
    auto operator()(T x) const { return x * x; }
};
```

## 🧩 第四部分：陷阱题 & 实现题（难度：★★★★）

### ✅ **题 9：悬空引用捕获**

```cpp
std::function f;

{
    int x = 42;
    f = [&x]() {
        std::cout << x << std::endl;
    };
}

f();  // 安全吗？
```


❌ 危险！x 生命周期已经结束，引用捕获导致 **悬空引用**。
### ✅ **题 10：手写模拟 Lambda 的类**

模拟这个 Lambda：

```cpp
int a = 1, b = 2;
auto f = [a, &b](int x) { return a + b + x; };
```

模拟实现如下：
```cpp
struct MyLambda {
    int a;
    int& b;

    MyLambda(int a_, int& b_) : a(a_), b(b_) {}

    int operator()(int x) const {
        return a + b + x;
    }
};
```