---
date: 2024-09-04T18:28:51+08:00
title: C语言中处理二维数组
description: C语言中处理二维数组
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
categories: Tutorials
---
## 前言

在C语言中，二维数组是一种常见的数据结构，用于存储和管理多维数据。
本篇博客将介绍如何在C语言中如何实现对结构体中的二维数组的处理，并提供具体的代码示例，作为c语言开发时的参考。

## 结构体中元素为普通数据的二维数组处理

### 示例代码

```c
#include <stdio.h>
#include <stdlib.h>

#define ROWS 5   // 二维数组的行数
#define COLS 5   // 二维数组的列数

// 定义一个结构体，包含一个二维数组
typedef struct {
    int matrix[ROWS][COLS];  // 二维数组
} Matrix;

// 初始化二维数组
void initializeMatrix(Matrix* m) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            m->matrix[i][j] = 0;  // 初始化所有元素为0
        }
    }
}

// 打印二维数组
void printMatrix(const Matrix* m) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            printf("%d ", m->matrix[i][j]);
        }
        printf("\n");
    }
}

// 插入一个值到二维数组的指定位置
void insertElement(Matrix* m, int row, int col, int value) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        m->matrix[row][col] = value;
    } else {
        printf("插入失败：索引超出范围。\n");
    }
}

// 删除指定位置的元素（将其设置为0）
void deleteElement(Matrix* m, int row, int col) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        m->matrix[row][col] = 0;
    } else {
        printf("删除失败：索引超出范围。\n");
    }
}

// 修改指定位置的元素
void updateElement(Matrix* m, int row, int col, int newValue) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        m->matrix[row][col] = newValue;
    } else {
        printf("修改失败：索引超出范围。\n");
    }
}

// 查询指定位置的元素
int getElement(const Matrix* m, int row, int col) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        return m->matrix[row][col];
    } else {
        printf("查询失败：索引超出范围。\n");
        return -1; // 返回一个错误值
    }
}

int main() {
    Matrix m;
    initializeMatrix(&m);  // 初始化矩阵

    printf("初始化后的矩阵：\n");
    printMatrix(&m);

    insertElement(&m, 2, 2, 10);  // 在 (2,2) 位置插入值 10
    printf("\n插入元素后的矩阵：\n");
    printMatrix(&m);

    updateElement(&m, 2, 2, 20);  // 修改 (2,2) 位置的元素为 20
    printf("\n修改元素后的矩阵：\n");
    printMatrix(&m);

    deleteElement(&m, 2, 2);  // 删除 (2,2) 位置的元素
    printf("\n删除元素后的矩阵：\n");
    printMatrix(&m);

    int value = getElement(&m, 2, 2);  // 查询 (2,2) 位置的元素
    printf("\n查询 (2,2) 位置的元素值：%d\n", value);

    return 0;
}
```
### 说明

1. **结构体定义**：`Matrix`结构体包含一个`matrix`二维数组。
2. **初始化函数**：`initializeMatrix`函数将二维数组中的所有元素初始化为0。
3. **打印函数**：`printMatrix`函数用于打印二维数组的内容。
4. **增删改查函数**：分别为`insertElement`、`deleteElement`、`updateElement`和`getElement`函数，分别用于插入、删除、更新和查询二维数组中的元素。

## 结构体中元素为结构体的二维数组处理

### 示例代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ROWS 3   // 二维数组的行数
#define COLS 3   // 二维数组的列数

// 定义一个结构体，表示二维数组中的元素
typedef struct {
    int id;
    char name[50];
    float score;
} Element;

// 定义一个结构体，包含一个元素类型的二维数组
typedef struct {
    Element matrix[ROWS][COLS];  // 二维数组，元素为Element结构体
} Matrix;

// 初始化二维数组
void initializeMatrix(Matrix* m) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            m->matrix[i][j].id = 0;  // 初始化id为0
            strcpy(m->matrix[i][j].name, "");  // 初始化name为空字符串
            m->matrix[i][j].score = 0.0f;  // 初始化score为0.0
        }
    }
}

// 打印二维数组
void printMatrix(const Matrix* m) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            printf("(%d, %s, %.2f) ", m->matrix[i][j].id, m->matrix[i][j].name, m->matrix[i][j].score);
        }
        printf("\n");
    }
}

// 插入一个结构体到二维数组的指定位置
void insertElement(Matrix* m, int row, int col, int id, const char* name, float score) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        m->matrix[row][col].id = id;
        strcpy(m->matrix[row][col].name, name);
        m->matrix[row][col].score = score;
    } else {
        printf("插入失败：索引超出范围。\n");
    }
}

// 删除指定位置的结构体元素（将其重置为默认值）
void deleteElement(Matrix* m, int row, int col) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        m->matrix[row][col].id = 0;
        strcpy(m->matrix[row][col].name, "");
        m->matrix[row][col].score = 0.0f;
    } else {
        printf("删除失败：索引超出范围。\n");
    }
}

// 修改指定位置的结构体元素
void updateElement(Matrix* m, int row, int col, int newId, const char* newName, float newScore) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        m->matrix[row][col].id = newId;
        strcpy(m->matrix[row][col].name, newName);
        m->matrix[row][col].score = newScore;
    } else {
        printf("修改失败：索引超出范围。\n");
    }
}

// 查询指定位置的结构体元素
Element getElement(const Matrix* m, int row, int col) {
    Element emptyElement = {0, "", 0.0f};  // 用于返回空元素
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        return m->matrix[row][col];
    } else {
        printf("查询失败：索引超出范围。\n");
        return emptyElement;
    }
}

int main() {
    Matrix m;
    initializeMatrix(&m);  // 初始化矩阵

    printf("初始化后的矩阵：\n");
    printMatrix(&m);

    // 插入元素
    insertElement(&m, 1, 1, 1, "Alice", 88.5f);  
    printf("\n插入元素后的矩阵：\n");
    printMatrix(&m);

    // 修改元素
    updateElement(&m, 1, 1, 2, "Bob", 92.0f);  
    printf("\n修改元素后的矩阵：\n");
    printMatrix(&m);

    // 删除元素
    deleteElement(&m, 1, 1);  
    printf("\n删除元素后的矩阵：\n");
    printMatrix(&m);

    // 查询元素
    Element e = getElement(&m, 1, 1);
    printf("\n查询 (1, 1) 位置的元素：id=%d, name=%s, score=%.2f\n", e.id, e.name, e.score);

    return 0;
}
```
### 说明

1. **结构体定义**：
    - `Element` 结构体包含3个成员：`id`（整型）、`name`（字符串）和`score`（浮点型）。
    - `Matrix` 结构体包含一个 `Element` 类型的二维数组 `matrix`。
2. **初始化函数**：`initializeMatrix` 函数将二维数组中的所有元素初始化为默认值（`id` 为0，`name`为空字符串，`score`为0.0）。
    
3. **打印函数**：`printMatrix` 函数用于打印整个二维数组的内容。
    
4. **增删改查函数**：
    - `insertElement` 函数用于插入一个新的 `Element` 到指定位置。
    - `deleteElement` 函数用于删除指定位置的元素（将其重置为默认值）。
    - `updateElement` 函数用于修改指定位置的元素。
    - `getElement` 函数用于查询并返回指定位置的 `Element`。
## 结构体中元素为结构体且结构体中包含指针元素的二维数组处理

### 示例代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ROWS 3  // 二维数组的行数
#define COLS 3  // 二维数组的列数

// 定义一个结构体，表示二维数组中的元素
typedef struct {
    int id;
    char* name;  // 动态分配的字符串指针
    float score;
} Element;

// 定义一个结构体，包含一个元素类型的二维数组
typedef struct {
    Element matrix[ROWS][COLS];  // 二维数组，元素为Element结构体
} Matrix;

// 初始化二维数组
void initializeMatrix(Matrix* m) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            m->matrix[i][j].id = 0;  // 初始化id为0
            m->matrix[i][j].name = NULL;  // 初始化指针为空
            m->matrix[i][j].score = 0.0f;  // 初始化score为0.0
        }
    }
}

// 打印二维数组
void printMatrix(const Matrix* m) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            printf("(%d, %s, %.2f) ", m->matrix[i][j].id,
                m->matrix[i][j].name ? m->matrix[i][j].name : "NULL",
                m->matrix[i][j].score);
        }
        printf("\n");
    }
}

// 插入一个结构体到二维数组的指定位置
void insertElement(Matrix* m, int row, int col, int id, const char* name, float score) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        // 释放之前的内存（如果有）
        if (m->matrix[row][col].name) {
            free(m->matrix[row][col].name);
        }
        m->matrix[row][col].id = id;
        m->matrix[row][col].name = malloc(strlen(name) + 1);  // 分配内存
        if (m->matrix[row][col].name) {
            strcpy(m->matrix[row][col].name, name);  // 复制字符串
        }
        m->matrix[row][col].score = score;
    } else {
        printf("插入失败：索引超出范围。\n");
    }
}

// 删除指定位置的结构体元素（释放内存并重置为默认值）
void deleteElement(Matrix* m, int row, int col) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        m->matrix[row][col].id = 0;
        if (m->matrix[row][col].name) {
            free(m->matrix[row][col].name);  // 释放动态分配的内存
            m->matrix[row][col].name = NULL;
        }
        m->matrix[row][col].score = 0.0f;
    } else {
        printf("删除失败：索引超出范围。\n");
    }
}

// 修改指定位置的结构体元素
void updateElement(Matrix* m, int row, int col, int newId, const char* newName, float newScore) {
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        // 释放之前的内存（如果有）
        if (m->matrix[row][col].name) {
            free(m->matrix[row][col].name);
        }
        m->matrix[row][col].id = newId;
        m->matrix[row][col].name = malloc(strlen(newName) + 1);  // 分配内存
        if (m->matrix[row][col].name) {
            strcpy(m->matrix[row][col].name, newName);  // 复制新字符串
        }
        m->matrix[row][col].score = newScore;
    } else {
        printf("修改失败：索引超出范围。\n");
    }
}

// 查询指定位置的结构体元素
Element getElement(const Matrix* m, int row, int col) {
    Element emptyElement = {0, NULL, 0.0f};  // 用于返回空元素
    if (row >= 0 && row < ROWS && col >= 0 && col < COLS) {
        return m->matrix[row][col];
    } else {
        printf("查询失败：索引超出范围。\n");
        return emptyElement;
    }
}

// 释放矩阵中所有动态分配的内存
void freeMatrix(Matrix* m) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            if (m->matrix[i][j].name) {
                free(m->matrix[i][j].name);
                m->matrix[i][j].name = NULL;
            }
        }
    }
}

int main() {
    Matrix m;
    initializeMatrix(&m);  // 初始化矩阵

    printf("初始化后的矩阵：\n");
    printMatrix(&m);

    // 插入元素
    insertElement(&m, 1, 1, 1, "Alice", 88.5f);  
    printf("\n插入元素后的矩阵：\n");
    printMatrix(&m);

    // 修改元素
    updateElement(&m, 1, 1, 2, "Bob", 92.0f);  
    printf("\n修改元素后的矩阵：\n");
    printMatrix(&m);

    // 删除元素
    deleteElement(&m, 1, 1);  
    printf("\n删除元素后的矩阵：\n");
    printMatrix(&m);

    // 查询元素
    Element e = getElement(&m, 1, 1);
    printf("\n查询 (1, 1) 位置的元素：id=%d, name=%s, score=%.2f\n", e.id, e.name ? e.name : "NULL", e.score);

    // 释放矩阵中的所有动态分配的内存
    freeMatrix(&m);

    return 0;
}
```
### 说明

1. **结构体定义**：
    
    - `Element` 结构体包含三个成员：`id`（整型）、`name`（指向字符数组的指针）和`score`（浮点型）。
    - `Matrix` 结构体包含一个 `Element` 类型的二维数组 `matrix`。
2. **内存管理**：
    
    - 每次插入或修改 `name` 字段时，动态分配内存（使用 `malloc`），并将字符串复制到分配的内存中。
    - 在插入、修改或删除元素前，先检查是否已有动态分配的内存，如果有则使用 `free` 释放。
    - 提供了 `freeMatrix` 函数，用于在程序结束时释放所有动态分配的内存，防止内存泄漏。
3. **增删改查操作**：
    
    - `insertElement`、`updateElement`、`deleteElement` 和 `getElement` 函数分别实现增、删、改、查操作，同时正确地处理内存分配和释放。

## 动态生成包含指针的结构体的二维数组

### 示例代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 定义一个结构体，表示二维数组中的元素
typedef struct {
    int id;
    char* name;  // 动态分配的字符串指针
    float score;
} Element;

// 定义一个结构体，用来管理动态生成的二维数组
typedef struct {
    int rows;
    int cols;
    Element** matrix;  // 指向指针的指针，表示动态分配的二维数组
} Matrix;

// 初始化动态二维数组
Matrix* createMatrix(int rows, int cols) {
    Matrix* m = (Matrix*)malloc(sizeof(Matrix));  // 为Matrix结构体分配内存
    if (m == NULL) {
        printf("内存分配失败。\n");
        return NULL;
    }

    m->rows = rows;
    m->cols = cols;
    m->matrix = (Element**)malloc(rows * sizeof(Element*));  // 分配行指针数组
    if (m->matrix == NULL) {
        printf("内存分配失败。\n");
        free(m);
        return NULL;
    }

    for (int i = 0; i < rows; i++) {
        m->matrix[i] = (Element*)malloc(cols * sizeof(Element));  // 为每行分配内存
        if (m->matrix[i] == NULL) {
            printf("内存分配失败。\n");
            // 释放已分配的内存
            for (int j = 0; j < i; j++) {
                free(m->matrix[j]);
            }
            free(m->matrix);
            free(m);
            return NULL;
        }

        // 初始化每个元素
        for (int j = 0; j < cols; j++) {
            m->matrix[i][j].id = 0;
            m->matrix[i][j].name = NULL;  // 初始化指针为空
            m->matrix[i][j].score = 0.0f;
        }
    }

    return m;
}

// 打印二维数组
void printMatrix(const Matrix* m) {
    for (int i = 0; i < m->rows; i++) {
        for (int j = 0; j < m->cols; j++) {
            printf("(%d, %s, %.2f) ", m->matrix[i][j].id,
                   m->matrix[i][j].name ? m->matrix[i][j].name : "NULL",
                   m->matrix[i][j].score);
        }
        printf("\n");
    }
}

// 插入一个结构体到二维数组的指定位置
void insertElement(Matrix* m, int row, int col, int id, const char* name, float score) {
    if (row >= 0 && row < m->rows && col >= 0 && col < m->cols) {
        // 释放之前的内存（如果有）
        if (m->matrix[row][col].name) {
            free(m->matrix[row][col].name);
        }
        m->matrix[row][col].id = id;
        m->matrix[row][col].name = malloc(strlen(name) + 1);  // 分配内存
        if (m->matrix[row][col].name) {
            strcpy(m->matrix[row][col].name, name);  // 复制字符串
        }
        m->matrix[row][col].score = score;
    } else {
        printf("插入失败：索引超出范围。\n");
    }
}

// 删除指定位置的结构体元素（释放内存并重置为默认值）
void deleteElement(Matrix* m, int row, int col) {
    if (row >= 0 && row < m->rows && col >= 0 && col < m->cols) {
        m->matrix[row][col].id = 0;
        if (m->matrix[row][col].name) {
            free(m->matrix[row][col].name);  // 释放动态分配的内存
            m->matrix[row][col].name = NULL;
        }
        m->matrix[row][col].score = 0.0f;
    } else {
        printf("删除失败：索引超出范围。\n");
    }
}

// 修改指定位置的结构体元素
void updateElement(Matrix* m, int row, int col, int newId, const char* newName, float newScore) {
    if (row >= 0 && row < m->rows && col >= 0 && col < m->cols) {
        // 释放之前的内存（如果有）
        if (m->matrix[row][col].name) {
            free(m->matrix[row][col].name);
        }
        m->matrix[row][col].id = newId;
        m->matrix[row][col].name = malloc(strlen(newName) + 1);  // 分配内存
        if (m->matrix[row][col].name) {
            strcpy(m->matrix[row][col].name, newName);  // 复制新字符串
        }
        m->matrix[row][col].score = newScore;
    } else {
        printf("修改失败：索引超出范围。\n");
    }
}

// 查询指定位置的结构体元素
Element getElement(const Matrix* m, int row, int col) {
    Element emptyElement = {0, NULL, 0.0f};  // 用于返回空元素
    if (row >= 0 && row < m->rows && col >= 0 && col < m->cols) {
        return m->matrix[row][col];
    } else {
        printf("查询失败：索引超出范围。\n");
        return emptyElement;
    }
}

// 释放矩阵中所有动态分配的内存
void freeMatrix(Matrix* m) {
    if (!m) return;
    
    for (int i = 0; i < m->rows; i++) {
        for (int j = 0; j < m->cols; j++) {
            if (m->matrix[i][j].name) {
                free(m->matrix[i][j].name);
            }
        }
        free(m->matrix[i]);  // 释放每一行的内存
    }
    free(m->matrix);  // 释放行指针数组
    free(m);  // 释放Matrix结构体本身
}

int main() {
    int rows = 3, cols = 3;
    Matrix* m = createMatrix(rows, cols);  // 动态创建矩阵

    if (m == NULL) {
        printf("矩阵创建失败。\n");
        return 1;
    }

    printf("初始化后的矩阵：\n");
    printMatrix(m);

    // 插入元素
    insertElement(m, 1, 1, 1, "Alice", 88.5f);
    printf("\n插入元素后的矩阵：\n");
    printMatrix(m);

    // 修改元素
    updateElement(m, 1, 1, 2, "Bob", 92.0f);
    printf("\n修改元素后的矩阵：\n");
    printMatrix(m);

    // 删除元素
    deleteElement(m, 1, 1);
    printf("\n删除元素后的矩阵：\n");
    printMatrix(m);

    // 查询元素
    Element e = getElement(m, 1, 1);
    printf("\n查询 (1, 1) 位置的元素：id=%d, name=%s, score=%.2f\n", e.id, e.name ? e.name : "NULL", e.score);

    // 释放矩阵中的所有动态分配的内存
    freeMatrix(m);

    return 0;
}
```
### 说明

1. **结构体定义**：
    
    - `Element` 结构体包含三个成员：`id`（整型）、`name`（指向动态分配的字符数组的指针）、`score`（浮点型）。
    - `Matrix` 结构体包含一个动态二维数组的指针 `matrix`、行数和列数。
2. **内存管理**：
    
    - `createMatrix` 函数动态分配内存来创建一个 `Matrix` 结构体和二维数组。首先分配 `Matrix` 结构体，然后为每一行指针数组和每个元素分配内存。
    - 在插入或修改元素时，动态分配内存给 `name` 指针，并使用 `free` 释放之前的内存。
    - `freeMatrix` 函数用于在程序结束时释放所有动态分配的内存，防止内存泄漏。
3. **增删改查操作**：
    
    - `insertElement`、`updateElement`、`deleteElement` 和 `getElement` 函数分别实现增、删、改、查操作，并正确地处理内存分配和释放。


## 二维数组行列数需要随数据规模变化而变化
实话说，要实现这个功能，对于纯c语言真的麻烦。
### 示例代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 定义一个结构体，表示二维数组中的元素
typedef struct {
    int id;
    char* name;  // 动态分配的字符串指针
    float score;
} Element;

// 定义一个结构体，用来管理动态生成的二维数组
typedef struct {
    int rows;       // 当前行数
    int cols;       // 当前列数
    int capacity;   // 当前容量（列的最大值）
    Element** matrix;  // 指向指针的指针，表示动态分配的二维数组
} DynamicMatrix;

// 初始化动态二维数组
DynamicMatrix* createMatrix(int initialRows, int initialCols) {
    DynamicMatrix* m = (DynamicMatrix*)malloc(sizeof(DynamicMatrix));  // 为DynamicMatrix结构体分配内存
    if (m == NULL) {
        printf("内存分配失败。\n");
        return NULL;
    }

    m->rows = initialRows;
    m->cols = initialCols;
    m->capacity = initialCols;
    m->matrix = (Element**)malloc(initialRows * sizeof(Element*));  // 分配行指针数组
    if (m->matrix == NULL) {
        printf("内存分配失败。\n");
        free(m);
        return NULL;
    }

    for (int i = 0; i < initialRows; i++) {
        m->matrix[i] = (Element*)malloc(initialCols * sizeof(Element));  // 为每行分配内存
        if (m->matrix[i] == NULL) {
            printf("内存分配失败。\n");
            // 释放已分配的内存
            for (int j = 0; j < i; j++) {
                free(m->matrix[j]);
            }
            free(m->matrix);
            free(m);
            return NULL;
        }

        // 初始化每个元素
        for (int j = 0; j < initialCols; j++) {
            m->matrix[i][j].id = 0;
            m->matrix[i][j].name = NULL;  // 初始化指针为空
            m->matrix[i][j].score = 0.0f;
        }
    }

    return m;
}

// 扩展矩阵容量（增加列数）
void expandMatrix(DynamicMatrix* m) {
    int newCapacity = m->capacity * 2;  // 增加容量（例如，增加一倍）

    for (int i = 0; i < m->rows; i++) {
        Element* newRow = (Element*)realloc(m->matrix[i], newCapacity * sizeof(Element));
        if (newRow == NULL) {
            printf("内存重新分配失败。\n");
            return;
        }
        // 初始化新增加的元素
        for (int j = m->capacity; j < newCapacity; j++) {
            newRow[j].id = 0;
            newRow[j].name = NULL;
            newRow[j].score = 0.0f;
        }
        m->matrix[i] = newRow;
    }
    m->capacity = newCapacity;
}

// 添加新元素到二维数组中（按行扩展）
void addElement(DynamicMatrix* m, int row, int id, const char* name, float score) {
    if (row >= m->rows) {
        // 增加行数
        int newRows = row + 1;
        Element** newMatrix = (Element**)realloc(m->matrix, newRows * sizeof(Element*));
        if (newMatrix == NULL) {
            printf("内存重新分配失败。\n");
            return;
        }

        m->matrix = newMatrix;

        // 为新增的行分配内存
        for (int i = m->rows; i < newRows; i++) {
            m->matrix[i] = (Element*)malloc(m->capacity * sizeof(Element));
            if (m->matrix[i] == NULL) {
                printf("内存分配失败。\n");
                return;
            }

            // 初始化新行的每个元素
            for (int j = 0; j < m->capacity; j++) {
                m->matrix[i][j].id = 0;
                m->matrix[i][j].name = NULL;
                m->matrix[i][j].score = 0.0f;
            }
        }

        m->rows = newRows;
    }

    // 当列超过当前容量时扩展容量
    if (m->cols >= m->capacity) {
        expandMatrix(m);
    }

    // 插入元素
    m->matrix[row][m->cols].id = id;
    m->matrix[row][m->cols].name = malloc(strlen(name) + 1);  // 分配内存
    if (m->matrix[row][m->cols].name) {
        strcpy(m->matrix[row][m->cols].name, name);  // 复制字符串
    }
    m->matrix[row][m->cols].score = score;
    m->cols++;  // 增加列数
}

// 打印二维数组
void printMatrix(const DynamicMatrix* m) {
    for (int i = 0; i < m->rows; i++) {
        for (int j = 0; j < m->cols; j++) {
            printf("(%d, %s, %.2f) ", m->matrix[i][j].id,
                   m->matrix[i][j].name ? m->matrix[i][j].name : "NULL",
                   m->matrix[i][j].score);
        }
        printf("\n");
    }
}

// 释放矩阵中所有动态分配的内存
void freeMatrix(DynamicMatrix* m) {
    if (!m) return;

    for (int i = 0; i < m->rows; i++) {
        for (int j = 0; j < m->capacity; j++) {
            if (m->matrix[i][j].name) {
                free(m->matrix[i][j].name);
            }
        }
        free(m->matrix[i]);  // 释放每一行的内存
    }
    free(m->matrix);  // 释放行指针数组
    free(m);  // 释放DynamicMatrix结构体本身
}

int main() {
    int initialRows = 1, initialCols = 2;
    DynamicMatrix* m = createMatrix(initialRows, initialCols);  // 动态创建矩阵

    if (m == NULL) {
        printf("矩阵创建失败。\n");
        return 1;
    }

    printf("初始化后的矩阵：\n");
    printMatrix(m);

    // 添加新元素
    addElement(m, 0, 1, "Alice", 88.5f);
    addElement(m, 0, 2, "Bob", 92.0f);
    addElement(m, 1, 3, "Charlie", 75.0f);  // 自动扩展行
    printf("\n添加元素后的矩阵：\n");
    printMatrix(m);

    // 释放矩阵中的所有动态分配的内存
    freeMatrix(m);

    return 0;
}
```

### 关键实现说明

1. **动态扩展容量**：
    
    - 通过`realloc`函数来扩展二维数组的行数和列数。
    - 每次容量不足时，容量翻倍（或根据实际需求扩展），以避免频繁的内存分配操作。
2. **按需增加行或列**：
    
    - 如果需要插入新元素超过了当前容量，则调用`expandMatrix`函数扩展容量。
    - 当需要增加新行时，重新分配行指针数组，并为新行分配内存。
3. **内存管理**：
    
    - 确保在每次分配新内存之前，检查是否需要释放旧内存。
    - 提供`freeMatrix`函数用于释放所有动态分配的内存，防止内存泄漏。