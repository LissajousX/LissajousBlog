---
date: 2025-07-07T18:11:35+08:00
title: priority_queue完全指南
description: priority_queue完全指南
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - cpp11
categories: Tutorials
---

# 🌟 C++ `std::priority_queue` 完全指南

`std::priority_queue` 是 C++ STL 提供的**优先队列容器适配器**，底层默认以 **最大堆（大顶堆）** 实现，可支持自定义排序方式与结构体，非常适合处理任务调度、图最短路径等场景。

---

## 📌 基本概念

- 优先队列中的元素总是**按优先级自动排序**。
    
- 默认情况下，是**最大堆**，即堆顶元素是**最大值**。
    
- 底层实现是用 `std::vector` + `make_heap` / `push_heap` 等堆操作实现的。
    

---

## 🧱 定义与初始化方式

### ✅ 默认最大堆（大顶堆）

```cpp
std::priority_queue<int> pq;  // 默认大顶堆，最大值优先
```

### ✅ 最小堆（小顶堆）

```cpp
#include <functional> // std::greater
std::priority_queue<int, std::vector<int>, std::greater<int>> pq;
```

### ✅ 初始化时传入已有数据

```cpp
std::vector<int> data = {3, 1, 4, 1, 5};
std::priority_queue<int> pq(data.begin(), data.end());
```

---

## 🧰 常用 API

|函数|含义与说明|
|---|---|
|`push(const T& val)`|插入元素|
|`pop()`|删除堆顶元素（优先级最高）|
|`top()`|返回堆顶元素（不删除）|
|`empty()`|判断是否为空|
|`size()`|返回元素数量|
|`emplace(args...)`|原地构造元素（效率更高）|
|`swap(other)`|与其他 priority_queue 交换内容|

---

## 🧪 示例：最大堆

```cpp
#include <iostream>
#include <queue>

int main() {
    std::priority_queue<int> pq;

    pq.push(5);
    pq.push(2);
    pq.push(8);

    while (!pq.empty()) {
        std::cout << pq.top() << " ";  // 输出：8 5 2
        pq.pop();
    }
}
```

---

## 🧪 示例：最小堆

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <functional>

int main() {
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;

    pq.push(5);
    pq.push(2);
    pq.push(8);

    while (!pq.empty()) {
        std::cout << pq.top() << " ";  // 输出：2 5 8
        pq.pop();
    }
}
```

---

## 🧱 自定义结构体与比较器

### 📦 场景：任务调度系统，按任务优先级排序

```cpp
#include <iostream>
#include <queue>
#include <vector>

struct Task {
    int id;
    int priority;
};

struct cmp {
    bool operator()(const Task& a, const Task& b) {
        return a.priority < b.priority; // priority大的排在前面
    }
};

int main() {
    std::priority_queue<Task, std::vector<Task>, cmp> tasks;

    tasks.push({1, 10});
    tasks.push({2, 5});
    tasks.push({3, 20});

    while (!tasks.empty()) {
        std::cout << "Task ID: " << tasks.top().id << ", Priority: " << tasks.top().priority << "\n";
        tasks.pop();
    }
}
```

📝 **输出：**

```
Task ID: 3, Priority: 20
Task ID: 1, Priority: 10
Task ID: 2, Priority: 5
```

---

## 📎 小技巧总结

- 默认是最大堆（`std::less`），若想用最小堆用 `std::greater`。
    
- 自定义排序时一定要重载仿函数（即比较器结构体）。
    
- 使用结构体时记得提供比较器，否则会报错。
    
- `emplace()` 比 `push()` 效率更高，推荐在性能敏感场合使用。
    

---

## 📚 适用场景

|场景|原因|
|---|---|
|Top K 问题|可使用最小堆维护前 K 大元素|
|图的最短路径算法|Dijkstra 算法中维护最短路径表|
|贪心调度问题|优先处理优先级高的任务|
|模拟事件优先级处理器|如多线程任务调度、定时器事件管理等|
