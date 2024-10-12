---
date: 2024-10-12T15:27:15+08:00
title: Python函数式编程
description: Python函数式编程
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - python
categories: Tutorials
---
### 1. 函数式编程简介
函数式编程是一种编程范式，强调使用函数作为基本构建块。与命令式编程相比，它更关注于表达计算的逻辑而不是步骤。

### 2. 高阶函数
高阶函数是接受函数作为参数或返回函数的函数。

**示例：**
```python
def square(x):
    return x * x

def apply_function(func, value):
    return func(value)

result = apply_function(square, 5)  # 输出: 25
```

### 3. 匿名函数
匿名函数使用 `lambda` 表达式定义，无需命名。

**示例：**
```python
add = lambda x, y: x + y
result = add(3, 5)  # 输出: 8
```

### 4. 不可变数据结构
不可变数据结构如元组，确保数据不被修改。

**示例：**
```python
coordinates = (10, 20)
# coordinates[0] = 15  # 会报错，元组不可变
```

### 5. 闭包
闭包是一个函数，其内部引用了外部函数的变量。

**示例：**
```python
def outer_function(x):
    def inner_function(y):
        return x + y
    return inner_function

closure = outer_function(10)
result = closure(5)  # 输出: 15
```

### 6. 装饰器
装饰器用于在不修改函数代码的情况下增强其功能。

**示例：**
```python
def decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@decorator
def say_hello():
    print("Hello!")

say_hello()
```

### 7. 函数组合
函数组合是将多个函数合并为一个函数的过程。

**示例：**
```python
def add_one(x):
    return x + 1

def multiply_by_two(x):
    return x * 2

def combined_function(x):
    return multiply_by_two(add_one(x))

result = combined_function(3)  # 输出: 8
```

### 8. 柯里化
柯里化是将多个参数的函数转换为一系列只接受一个参数的函数。

**示例：**
```python
def curried_add(x):
    def add_y(y):
        return x + y
    return add_y

add_five = curried_add(5)
result = add_five(10)  # 输出: 15
```

