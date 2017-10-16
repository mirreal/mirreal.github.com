---
layout: post
title: "使用双端队列模拟动态数组的部分功能"
date: 2014-07-02 15:45:05 +0800
comments: true
categories:
---


C语言作为一门古老的语言，在系统级，高性能的领域依然独领风骚，但有些时候，使用起来总是没那么顺手。

比如数组。这里使用双端队列简单的实现了一些类似动态数组的功能。

模拟 stack， queue 实现 push， pop， shift， unshift 操作，以及两个遍历方法，for_each() 和 map()，其中 for_each 接受一个函数，函数包括一个 item 参数，map 方法与其类似，但是返回一个数组。

<!-- more -->

#### Double Queue 定义：

```c
typedef int data;
typedef struct node* node;
typedef struct double_queue* dqueue;
struct node {
  data item;
  node prev, next;
};
struct double_queue {
  int count;
  node head, tail;
};
```

#### Push

```c
void dq_push(dqueue dq, data item) {
  node t = malloc(sizeof *t);
  t->item = item;
  t->prev = dq->tail;
  t->next = NULL;
  if (dq->count) {
    dq->tail->next = t;
  } else {
    dq->head = t;
  }
  dq->tail= t;
  dq->count++;
}
```

#### pop

```c
data dq_pop(dqueue dq) {
  data item = dq->tail->item;
  dq->tail->prev->next = NULL;
  dq->tail = dq->tail->prev;
  dq->count--;
  return item;
}
```

#### shift

```c
data dq_shift(dqueue dq) {
  data item = dq->head->item;
  dq->head->next->prev = NULL;
  dq->head = dq->head->next;
  dq->count--;
  return item;
}
```

#### unshift

```c
void dq_unshift(dqueue dq, data item) {
  node t = malloc(sizeof *t);
  t->item = item;
  t->prev = NULL;
  t->next = dq->head;
  if (dq->count) {
    dq->head->prev = t;
  } else {
    dq->tail = t;
  }
  dq->head = t;
  dq->count++;
}
```

#### 遍历操作：

##### for_each

```c
void dq_for_each(dqueue dq, void f(data)) {
  node t = malloc(sizeof *t);
  for (t = dq->head; t != NULL; t = t->next) {
    f(t->item);
  }
}
```

##### map

```c
int* dq_map(dqueue dq, data f(data)) {
  int *a = malloc(dq->count * sizeof(int)), i;
  node t = malloc(sizeof *t);
  for (t = dq->head, i = 0; t != NULL; t = t->next, i++) {
    a[i] = f(t->item);
  }
  return a;
}
```


#### 完整源代码及测试用例：


```c double_queue.c

#include <stdio.h>
#include <stdlib.h>

typedef int data;
typedef struct node* node;
typedef struct double_queue* dqueue;
struct node {
  data item;
  node prev, next;
};
struct double_queue {
  int count;
  node head, tail;
};

void dq_init(dqueue dq) {
  dq->count = 0;
  dq->head = dq->tail = NULL;
}

int dq_empty(dqueue dq) {
  return dq->count == 0;
}

//add item from head
void dq_unshift(dqueue dq, data item) {
  node t = malloc(sizeof *t);
  t->item = item;
  t->prev = NULL;
  t->next = dq->head;
  if (dq->count) {
	dq->head->prev = t;
  } else {
	dq->tail = t;
  }
  dq->head = t;
  dq->count++;
}

//add item from tail
void dq_push(dqueue dq, data item) {
  node t = malloc(sizeof *t);
  t->item = item;
  t->prev = dq->tail;
  t->next = NULL;
  if (dq->count) {
	dq->tail->next = t;
  } else {
	dq->head = t;
  }
  dq->tail= t;
  dq->count++;
}

//delete first item
data dq_shift(dqueue dq) {
  data item = dq->head->item;
  dq->head->next->prev = NULL;
  dq->head = dq->head->next;
  dq->count--;
  return item;
}

//delete last item
data dq_pop(dqueue dq) {
  data item = dq->tail->item;
  dq->tail->prev->next = NULL;
  dq->tail = dq->tail->prev;
  dq->count--;
  return item;
}

void dq_for_each(dqueue dq, void f(data)) {
  node t = malloc(sizeof *t);
  for (t = dq->head; t != NULL; t = t->next) {
	f(t->item);
  }
}

int* dq_map(dqueue dq, data f(data)) {
  int *a = malloc(dq->count * sizeof(int)), i;
  node t = malloc(sizeof *t);
  for (t = dq->head, i = 0; t != NULL; t = t->next, i++) {
	a[i] = f(t->item);
  }
  return a;
}

void f(data item) {
  printf("%d ", item);
}

data f2(data item) {
  return 10*item;
}

void array_for_each(int a[]) {
}

int main() {
  int i;
  dqueue dq = malloc(sizeof *dq);
  dq_init(dq);
  for (i = 0; i < 4; i++) {
	dq_push(dq, 3*i+1);
  }
  printf("count: %d\n", dq->count);
  dq_for_each(dq, f);
  printf("\n");
  int *a = dq_map(dq, f2);
  for (i = 0; i < dq->count; i++) {
	printf("%d ", a[i]);
  }
  printf("\n");
  data tail = dq_pop(dq);
  printf("delete tail: %d\n", tail);
  data head = dq_shift(dq);
  printf("delete head: %d\n", head);
  printf("count: %d\n", dq->count);
  dq_for_each(dq, f);
  printf("\n");
  return 0;
}
```
