---
layout: page
title: "Josephus Problem"
date: 2014-7-8 21:55
comments: true
sharing: true
footer: true
---

#### July 8th, 2014

## linked list

### C

```c
struct node {
  item item;
  struct node* next;
}

struct node* head = malloc(sizeof(*head));
head->item = 1;
head->next = head;
```

#### C full-edition

```c josephus_problem.c
#include <stdio.h>
#include <stdlib.h>

typedef struct node* link;
struct node {
  int item;
  link  next;
};

int main(int argc, char *argv[]) {
  int i, N = atoi(argv[1]), M = atoi(argv[2]);
  link head = malloc(sizeof(struct node)), x;
  head->item = 1;
  head->next = head;

  for (i = 2, x = head; i <= N; i++) {
    x = (x->next = malloc(sizeof *x));
    x->item = i;
  }
  x->next = head;

  while(x != x->next) {
    for (i = 1; i < M; i++) x = x->next;
    printf("%d ", x->next->item);
    x->next = x->next->next;
  }

  printf("\nLeft: %d\n", x->item);
}
```


### Javascript

construct function

```javascript
function Node(item, next) {
  this.item = item;
  this.next = next;
}

var head = new Node(1);
head.next = head;
```

### Python

```python
class Node():
  def __init__(self, item, next):
    self.item = item
    self.next = mext
  }

head = Node(1)
head.next = head
```

### Java

```java
class Node {
  int item;
  Node next;
}

Node head = new Node();
head.item = 1;
head.next = head;
```
