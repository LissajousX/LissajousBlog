---
date: 2024-09-06T11:38:27+08:00
title: C语言链表实现
description: C语言链表实现
featured_image: /images/book.png
images:
  - /images/book.png
tags:
  - tutorial
  - C
categories: Tutorials
---
# 单链表
单链表是最基本的链表，每个节点包含一个数据域和一个指向下一个节点的指针。

```c
#include <stdio.h>
#include <stdlib.h>

// 单链表节点定义
struct SinglyNode {
    int data;
    struct SinglyNode* next;
};

// 创建新单链表节点
struct SinglyNode* createSinglyNode(int data) {
    struct SinglyNode* newNode = (struct SinglyNode*)malloc(sizeof(struct SinglyNode));
    newNode->data = data;
    newNode->next = NULL;
    return newNode;
}

// 在单链表头部插入节点
void insertAtHeadSingly(struct SinglyNode** head, int data) {
    struct SinglyNode* newNode = createSinglyNode(data);
    newNode->next = *head;
    *head = newNode;
}

// 在单链表尾部插入节点
void insertAtTailSingly(struct SinglyNode** head, int data) {
    struct SinglyNode* newNode = createSinglyNode(data);
    if (*head == NULL) {
        *head = newNode;
        return;
    }

    struct SinglyNode* temp = *head;
    while (temp->next != NULL) {
        temp = temp->next;
    }
    temp->next = newNode;
}

// 在单链表的指定位置插入节点
void insertAtPositionSingly(struct SinglyNode** head, int data, int position) {
    if (position == 1) {
        insertAtHeadSingly(head, data);
        return;
    }

    struct SinglyNode* newNode = createSinglyNode(data);
    struct SinglyNode* temp = *head;

    for (int i = 1; i < position - 1 && temp != NULL; i++) {
        temp = temp->next;
    }

    if (temp == NULL) {
        printf("位置超出链表长度\n");
        return;
    }

    newNode->next = temp->next;
    temp->next = newNode;
}

// 删除单链表头部节点
void deleteHeadSingly(struct SinglyNode** head) {
    if (*head == NULL) return;

    struct SinglyNode* temp = *head;
    *head = (*head)->next;
    free(temp);
}

// 删除单链表尾部节点
void deleteTailSingly(struct SinglyNode** head) {
    if (*head == NULL) return;

    if ((*head)->next == NULL) {
        free(*head);
        *head = NULL;
        return;
    }

    struct SinglyNode* temp = *head;
    while (temp->next->next != NULL) {
        temp = temp->next;
    }

    free(temp->next);
    temp->next = NULL;
}

// 删除单链表的指定位置节点
void deleteAtPositionSingly(struct SinglyNode** head, int position) {
    if (*head == NULL) return;

    if (position == 1) {
        deleteHeadSingly(head);
        return;
    }

    struct SinglyNode* temp = *head;
    for (int i = 1; i < position - 1 && temp->next != NULL; i++) {
        temp = temp->next;
    }

    if (temp->next == NULL) {
        printf("位置超出链表长度\n");
        return;
    }

    struct SinglyNode* nodeToDelete = temp->next;
    temp->next = temp->next->next;
    free(nodeToDelete);
}

// 打印单链表
void printSinglyList(struct SinglyNode* head) {
    struct SinglyNode* current = head;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}
```
# 双链表
双链表中的每个节点都包含两个指针，一个指向下一个节点，另一个指向前一个节点，这使得我们可以在两个方向上遍历链表。
```c
// 双链表节点定义
struct DoublyNode {
    int data;
    struct DoublyNode* next;
    struct DoublyNode* prev;
};

// 创建新双链表节点
struct DoublyNode* createDoublyNode(int data) {
    struct DoublyNode* newNode = (struct DoublyNode*)malloc(sizeof(struct DoublyNode));
    newNode->data = data;
    newNode->next = NULL;
    newNode->prev = NULL;
    return newNode;
}

// 在双链表头部插入节点
void insertAtHeadDoubly(struct DoublyNode** head, int data) {
    struct DoublyNode* newNode = createDoublyNode(data);
    newNode->next = *head;
    if (*head != NULL) {
        (*head)->prev = newNode;
    }
    *head = newNode;
}

// 在双链表尾部插入节点
void insertAtTailDoubly(struct DoublyNode** head, int data) {
    struct DoublyNode* newNode = createDoublyNode(data);
    if (*head == NULL) {
        *head = newNode;
        return;
    }

    struct DoublyNode* temp = *head;
    while (temp->next != NULL) {
        temp = temp->next;
    }
    temp->next = newNode;
    newNode->prev = temp;
}

// 在双链表的指定位置插入节点
void insertAtPositionDoubly(struct DoublyNode** head, int data, int position) {
    if (position == 1) {
        insertAtHeadDoubly(head, data);
        return;
    }

    struct DoublyNode* newNode = createDoublyNode(data);
    struct DoublyNode* temp = *head;

    for (int i = 1; i < position - 1 && temp != NULL; i++) {
        temp = temp->next;
    }

    if (temp == NULL) {
        printf("位置超出链表长度\n");
        return;
    }

    newNode->next = temp->next;
    if (temp->next != NULL) {
        temp->next->prev = newNode;
    }
    newNode->prev = temp;
    temp->next = newNode;
}

// 删除双链表头部节点
void deleteHeadDoubly(struct DoublyNode** head) {
    if (*head == NULL) return;

    struct DoublyNode* temp = *head;
    *head = (*head)->next;

    if (*head != NULL) {
        (*head)->prev = NULL;
    }

    free(temp);
}

// 删除双链表尾部节点
void deleteTailDoubly(struct DoublyNode** head) {
    if (*head == NULL) return;

    if ((*head)->next == NULL) {
        free(*head);
        *head = NULL;
        return;
    }

    struct DoublyNode* temp = *head;
    while (temp->next != NULL) {
        temp = temp->next;
    }

    temp->prev->next = NULL;
    free(temp);
}

// 删除双链表的指定位置节点
void deleteAtPositionDoubly(struct DoublyNode** head, int position) {
    if (*head == NULL) return;

    if (position == 1) {
        deleteHeadDoubly(head);
        return;
    }

    struct DoublyNode* temp = *head;
    for (int i = 1; i < position && temp != NULL; i++) {
        temp = temp->next;
    }

    if (temp == NULL) {
        printf("位置超出链表长度\n");
        return;
    }

    if (temp->next != NULL) {
        temp->next->prev = temp->prev;
    }
    if (temp->prev != NULL) {
        temp->prev->next = temp->next;
    }

    free(temp);
}

// 打印双链表
void printDoublyList(struct DoublyNode* head) {
    struct DoublyNode* current = head;
    while (current != NULL) {
        printf("%d <-> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}
```
# 循环单链表
循环链表的最后一个节点指向链表的头节点，从而形成一个循环结构。
```c
// 循环链表节点定义（与单链表相同）
struct CircularNode {
    int data;
    struct CircularNode* next;
};

// 创建新循环链表节点
struct CircularNode* createCircularNode(int data) {
    struct CircularNode* newNode = (struct CircularNode*)malloc(sizeof(struct CircularNode));
    newNode->data = data;
    newNode->next = newNode; // 初始化时指向自己
    return newNode;
}

// 在循环链表头部插入节点
void insertAtHeadCircular(struct CircularNode** head, int data) {
    struct CircularNode* newNode = createCircularNode(data);
    if (*head == NULL) {
        *head = newNode;
    } else {
        struct CircularNode* temp = *head;
        while (temp->next != *head) {
            temp = temp->next;
        }
        temp->next = newNode;
        newNode->next = *head;
        *head = newNode;
    }
}

// 在循环链表尾部插入节点
void insertAtTailCircular(struct CircularNode** head, int data) {
    struct CircularNode* newNode = createCircularNode(data);
    if (*head == NULL) {
        *head = newNode;
        return;
    }

    struct CircularNode* temp = *head;
    while (temp->next != *head) {
        temp = temp->next;
    }
    temp->next = newNode;
    newNode->next = *head;
}

// 在循环链表的指定位置插入节点
void insertAtPositionCircular(struct CircularNode** head, int data, int position) {
    if (position == 1) {
        insertAtHeadCircular(head, data);
        return;
    }

    struct CircularNode* newNode = createCircularNode(data);
    struct CircularNode* temp = *head;

    for (int i = 1; i < position - 1 && temp->next != *head; i++) {
        temp = temp->next;
    }

    if (temp->next == *head) {
        printf("位置超出链表长度\n");
        return;
    }

    newNode->next = temp->next;
    temp->next = newNode;
}

// 删除循环链表头部节点
void deleteHeadCircular(struct CircularNode** head) {
    if (*head == NULL) return;

    struct CircularNode* temp = *head;
    struct CircularNode* last = *head;

    while (last->next != *head) {
        last = last->next;
    }

    if (*head == last) {
        free(*head);
        *head = NULL;
    } else {
        last->next = (*head)->next;
        *head = (*head)->next;
        free(temp);
    }
}

// 删除循环链表尾部节点
void deleteTailCircular(struct CircularNode** head) {
    if (*head == NULL) return;

    struct CircularNode* temp = *head;
    struct CircularNode* prev = NULL;

    while (temp->next != *head) {
        prev = temp;
        temp = temp->next;
    }

    if (temp == *head) {
        free(*head);
        *head = NULL;
    } else {
        prev->next = *head;
        free(temp);
    }
}

// 删除循环链表的指定位置节点
void deleteAtPositionCircular(struct CircularNode** head, int position) {
    if (*head == NULL) return;

    if (position == 1) {
        deleteHeadCircular(head);
        return;
    }

    struct CircularNode* temp = *head;
    struct CircularNode* prev = NULL;

    for (int i = 1; i < position && temp->next != *head; i++) {
        prev = temp;
        temp = temp->next;
    }

    if (temp->next == *head) {
        printf("位置超出链表长度\n");
        return;
    }

    prev->next = temp->next;
    free(temp);
}

// 打印循环链表
void printCircularList(struct CircularNode* head) {
    if (head == NULL) return;
    struct CircularNode* current = head;
    do {
        printf("%d -> ", current->data);
        current = current->next;
    } while (current != head);
    printf("(head)\n");
}
```
# 循环双链表
循环双链表不仅每个节点有前后指针，而且头节点和尾节点相连形成一个闭环。
```c
// 循环双链表节点定义
struct CircularDoublyNode {
    int data;
    struct CircularDoublyNode* next;
    struct CircularDoublyNode* prev;
};

// 创建新循环双链表节点
struct CircularDoublyNode* createCircularDoublyNode(int data) {
    struct CircularDoublyNode* newNode = (struct CircularDoublyNode*)malloc(sizeof(struct CircularDoublyNode));
    newNode->data = data;
    newNode->next = newNode;
    newNode->prev = newNode;
    return newNode;
}

// 在循环双链表头部插入节点
void insertAtHeadCircularDoubly(struct CircularDoublyNode** head, int data) {
    struct CircularDoublyNode* newNode = createCircularDoublyNode(data);
    if (*head == NULL) {
        *head = newNode;
    } else {
        struct CircularDoublyNode* last = (*head)->prev;
        newNode->next = *head;
        newNode->prev = last;
        last->next = newNode;
        (*head)->prev = newNode;
        *head = newNode;
    }
}

// 在循环双链表尾部插入节点
void insertAtTailCircularDoubly(struct CircularDoublyNode** head, int data) {
    struct CircularDoublyNode* newNode = createCircularDoublyNode(data);
    if (*head == NULL) {
        *head = newNode;
        return;
    }

    struct CircularDoublyNode* last = (*head)->prev;
    newNode->next = *head;
    newNode->prev = last;
    last->next = newNode;
    (*head)->prev = newNode;
}

// 在循环双链表的指定位置插入节点
void insertAtPositionCircularDoubly(struct CircularDoublyNode** head, int data, int position) {
    if (position == 1) {
        insertAtHeadCircularDoubly(head, data);
        return;
    }

    struct CircularDoublyNode* newNode = createCircularDoublyNode(data);
    struct CircularDoublyNode* temp = *head;

    for (int i = 1; i < position - 1 && temp->next != *head; i++) {
        temp = temp->next;
    }

    if (temp->next == *head) {
        printf("位置超出链表长度\n");
        return;
    }

    newNode->next = temp->next;
    newNode->prev = temp;
    temp->next->prev = newNode;
    temp->next = newNode;
}

// 删除循环双链表头部节点
void deleteHeadCircularDoubly(struct CircularDoublyNode** head) {
    if (*head == NULL) return;

    struct CircularDoublyNode* temp = *head;
    struct CircularDoublyNode* last = (*head)->prev;

    if (*head == (*head)->next) {
        free(*head);
        *head = NULL;
    } else {
        last->next = (*head)->next;
        (*head)->next->prev = last;
        *head = (*head)->next;
        free(temp);
    }
}

// 删除循环双链表尾部节点
void deleteTailCircularDoubly(struct CircularDoublyNode** head) {
    if (*head == NULL) return;

    struct CircularDoublyNode* last = (*head)->prev;
    struct CircularDoublyNode* secondLast = last->prev;

    if (last == *head) {
        free(*head);
        *head = NULL;
    } else {
        secondLast->next = *head;
        (*head)->prev = secondLast;
        free(last);
    }
}

// 删除循环双链表的指定位置节点
void deleteAtPositionCircularDoubly(struct CircularDoublyNode** head, int position) {
    if (*head == NULL) return;

    if (position == 1) {
        deleteHeadCircularDoubly(head);
        return;
    }

    struct CircularDoublyNode* temp = *head;
    for (int i = 1; i < position && temp->next != *head; i++) {
        temp = temp->next;
    }

    if (temp->next == *head) {
        printf("位置超出链表长度\n");
        return;
    }

    temp->prev->next = temp->next;
    temp->next->prev = temp->prev;
    free(temp);
}

// 打印循环双链表
void printCircularDoublyList(struct CircularDoublyNode* head) {
    if (head == NULL) return;
    struct CircularDoublyNode* current = head;
    do {
        printf("%d <-> ", current->data);
        current = current->next;
    } while (current != head);
    printf("(head)\n");
}
```