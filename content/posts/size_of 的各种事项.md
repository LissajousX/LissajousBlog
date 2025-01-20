---
date: 2025-01-20T10:02:39+08:00
title: size_of 的各种事项
description: size_of 的各种事项
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - C
categories: Tutorials
---
`sizeof` 是 C 和 C++ 中的一个操作符，用于计算数据类型或对象在内存中的大小（以字节为单位）。以下是它的用法和注意事项：

---

### **用法**

1. **计算基本数据类型的大小**
    
    ```c
    printf("Size of int: %zu\n", sizeof(int));
    printf("Size of char: %zu\n", sizeof(char));
    printf("Size of double: %zu\n", sizeof(double));
    ```
    
2. **计算结构体或类的大小**
    
    ```c
    struct MyStruct {
        int a;
        char b;
        double c;
    };
    printf("Size of MyStruct: %zu\n", sizeof(struct MyStruct));
    ```
    
3. **计算指针类型的大小**
    
    ```c
    int *ptr;
    printf("Size of pointer: %zu\n", sizeof(ptr));
    ```
    
4. **计算数组的大小**
    
    ```c
    int arr[10];
    printf("Size of array: %zu\n", sizeof(arr));
    printf("Number of elements in array: %zu\n", sizeof(arr) / sizeof(arr[0]));
    ```
    
5. **使用变量或表达式**
    
    ```c
    int x = 42;
    printf("Size of x: %zu\n", sizeof(x)); // Variable
    printf("Size of expression: %zu\n", sizeof(x + 1)); // Expression
    ```
    

---

### **注意事项**

1. **与数据类型的关系**
    
    - `sizeof` 返回的结果依赖于具体平台、编译器和目标架构。不同平台可能有不同的大小（如 32 位和 64 位系统）。
2. **计算数组大小**
    
    - `sizeof` 数组返回整个数组的大小，而不是数组的指针大小。
    - 如果数组退化为指针（如函数参数传递），`sizeof` 计算的是指针的大小，而不是数组本身。
    
    ```c
    void func(int arr[]) {
        printf("Size of arr in function: %zu\n", sizeof(arr)); // 输出的是指针大小
    }
    ```
    
3. **对动态分配的内存无效**
    
    - `sizeof` 不能直接用来计算动态分配的内存大小。例如，`malloc` 分配的内存需要用户自行管理。
    
    ```c
    int *p = malloc(10 * sizeof(int));
    printf("Size of p: %zu\n", sizeof(p)); // 仅返回指针大小
    ```
    
4. **结构体对齐**
    
    - 结构体的大小可能比成员变量的大小总和更大，这是因为内存对齐规则会引入填充字节。
5. **`sizeof` 是编译时操作**
    
    - 对于静态类型和静态分配的对象，`sizeof` 的结果在编译时计算。
    - 运行时动态分配的内存无法通过 `sizeof` 获取。
6. **括号的使用**
    
    - 对于类型名称，需要使用括号（`sizeof(type)`）。
    - 对于变量或表达式，可以不使用括号（`sizeof variable` 或 `sizeof expression`）。
7. **返回值类型**
    
    - `sizeof` 的返回类型是 `size_t`，通常定义为无符号整数类型。
8. **警惕对不完全类型使用 `sizeof`**
    
    - 如果使用 `sizeof` 对不完全类型（如声明但未定义的结构体）操作，会导致编译错误。
    
    ```c
    struct IncompleteType; // 声明未定义
    printf("Size: %zu\n", sizeof(struct IncompleteType)); // 错误
    ```
    

---

### **最佳实践**

- 对于跨平台代码，尽量避免对 `sizeof` 的结果作绝对假设，而应使用标准库宏（如 `INT_MAX` 等）或根据平台文档确认类型大小。
- 在使用动态分配内存时，建议始终使用 `sizeof` 来指定元素大小以提高代码可移植性。
    
    ```c
    int *p = (int *)malloc(10 * sizeof(*p)); // 推荐
    ```
    
---
### **错误使用示例**

以下是一些常见的 `sizeof` 错误使用示例，以及它们可能导致的问题和修正方法：

---

### **1. 数组退化为指针**

#### **错误示例**

```c
void printArraySize(int arr[]) {
    printf("Size of array: %zu\n", sizeof(arr)); // 实际输出的是指针的大小
}
int main() {
    int arr[10];
    printArraySize(arr); // 期待输出 40（10 * sizeof(int)），但实际是指针大小
    return 0;
}
```

#### **问题**

在函数参数中，数组退化为指针，因此 `sizeof(arr)` 返回的是指针大小，而不是数组大小。

#### **修正方法**

将数组大小作为额外参数传递：

```c
void printArraySize(int *arr, size_t size) {
    printf("Size of array: %zu\n", size);
}
int main() {
    int arr[10];
    printArraySize(arr, sizeof(arr)); // 传递数组大小
    return 0;
}
```

---

### **2. 动态分配内存的对象**

#### **错误示例**

```c
int *p = (int *)malloc(10 * sizeof(int));
printf("Size of allocated memory: %zu\n", sizeof(p)); // 返回的是指针大小
```

#### **问题**

`sizeof(p)` 只返回指针大小，而不是动态分配内存的大小。

#### **修正方法**

在内存分配时明确跟踪分配的大小：

```c
size_t allocated_size = 10 * sizeof(int);
printf("Size of allocated memory: %zu\n", allocated_size);
```

---

### **3. 结构体不完全类型**

#### **错误示例**

```c
struct IncompleteType;
printf("Size of IncompleteType: %zu\n", sizeof(struct IncompleteType)); // 错误：不完全类型
```

#### **问题**

不完全类型的大小在编译时未知，`sizeof` 无法操作。

#### **修正方法**

确保结构体已完整定义：

```c
struct CompleteType {
    int a;
    double b;
};
printf("Size of CompleteType: %zu\n", sizeof(struct CompleteType));
```

---

### **4. 忽略结构体对齐问题**

#### **错误示例**

```c
struct Misaligned {
    char c;
    int i;
    char d;
};
printf("Expected size: %zu\n", sizeof(char) + sizeof(int) + sizeof(char)); // 6
printf("Actual size: %zu\n", sizeof(struct Misaligned)); // 通常为 12 或 16（根据对齐规则）
```

#### **问题**

结构体的大小可能比成员大小总和更大，因内存对齐规则会插入填充字节。

#### **修正方法**

利用 `#pragma pack` 修改对齐方式（小心使用，可能影响性能）：

```c
#pragma pack(1) // 设置 1 字节对齐
struct Misaligned {
    char c;
    int i;
    char d;
};
#pragma pack() // 恢复默认对齐
printf("Size with pragma pack: %zu\n", sizeof(struct Misaligned));
```

---

### **5. 错误假设指针大小固定**

#### **错误示例**

```c
printf("Size of pointer: %zu\n", sizeof(void *)); // 假设总是 4 或 8
```

#### **问题**

指针大小依赖于目标平台（如 32 位系统上通常为 4 字节，64 位为 8 字节）。硬编码假设可能导致移植性问题。

#### **修正方法**

避免假设指针大小，使用 `sizeof` 获取：

```c
printf("Size of pointer: %zu\n", sizeof(void *)); // 运行时自动调整
```

---

### **6. 忽略表达式类型**

#### **错误示例**

```c
int x = 42;
printf("Size of expression: %zu\n", sizeof(x + 1.0)); // 返回 sizeof(double)
```

#### **问题**

`sizeof` 操作的是表达式的类型，而不是值。`x + 1.0` 的结果类型为 `double`，因此返回 `sizeof(double)`。

#### **修正方法**

确保明确表达式的类型：

```c
printf("Size of x as int: %zu\n", sizeof((int)(x + 1.0)));
```

---

### **7. 忘记括号**

#### **错误示例**

```c
printf("Size of int: %zu\n", sizeof int); // 错误：缺少括号
```

#### **问题**

当 `sizeof` 用于类型时，必须加括号。

#### **修正方法**

```c
printf("Size of int: %zu\n", sizeof(int)); // 正确
```

---

### **8. 忽略多维数组的总大小**

#### **错误示例**

```c
int matrix[3][4];
printf("Size of matrix row: %zu\n", sizeof(matrix[0])); // 正确
printf("Total size of matrix: %zu\n", sizeof(matrix) / sizeof(matrix[0])); // 忘记正确理解维度
```

#### **问题**

对于多维数组，`sizeof` 只针对某一维度。直接对数组本身操作可能导致混淆。

#### **修正方法**

清晰地解释每个维度：

```c
printf("Total size of matrix: %zu\n", sizeof(matrix)); // 整个矩阵大小
printf("Number of rows: %zu\n", sizeof(matrix) / sizeof(matrix[0])); // 行数
printf("Number of elements: %zu\n", sizeof(matrix) / sizeof(matrix[0][0])); // 总元素数
```
