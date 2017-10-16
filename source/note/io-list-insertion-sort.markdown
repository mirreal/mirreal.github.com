---
layout: page
title: "io & list insertion sort"
date: 2014-7-9 22:24
comments: true
sharing: true
footer: true
---


#### July 9th, 2014

##io

```c
int putchar(int c)
putchar(c) -- printf("%c", c) -- putc(int c, stdout)
int getchar() -- getc(stdin)
```

file version

```c
int putc(int c, FILE *stream)
int getc(FILE *stream)
```

EOF -- end-of-file

```c
#define EOF (-1)
```

## List Insertion sort

```c list_insertion_sort.c
#include <stdio.h>
#include <stdlib.h>
#define N 16

struct node {
  int item;
  struct node* next;
};
typedef struct node* link;

link reverse(link x);

int main() {
  struct node heada, headb;
  link t, u, x, a = &heada, b;
  int i;

  for (i = 0, t = a; i < N; i++) {
    t->next = malloc(sizeof(struct node));
    t = t->next;
    t->next = NULL;
    t->item = rand() % 1000;
  }

  b = &headb;
  b->next = NULL;

  for (t = a->next; t != NULL; t = u) {
    u = t->next;

    for (x = b; x->next != NULL; x = x->next) {
      if (x->next->item > t->item) break;
    }
    t->next = x->next;
    x->next = t;
  }

  for (t = b->next, i = 1; t != NULL; t = t-> next, i++) {
    printf("%3d: %3d ", i, t->item);
  }
  printf("\n");

  a = reverse(b);
  for (t = a, i = 1; t->next != NULL; t = t->next, i++) {
    printf("%3d: %3d ", i, t->item);
  }
  printf("\n");
}

link reverse(link x) {
  link y = x, t, r = NULL;
  while (y != NULL) {
    t = y->next;
    y->next = r;
    r = y;
    y = t;
  }
  return r;
}
```
