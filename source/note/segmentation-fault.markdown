---
layout: page
title: "segmentation fault & String method"
date: 2014-7-11 20:48
comments: true
sharing: true
footer: true
---

#### July 11th, 2014, Friday

## 1.Segmentation Fault

example 1:

```c
char *c;
strcpy(c, "cd");
```
mend:

```c
char *c = malloc(8 * sizeof(char));
strcpy(c, "cd");
```
or

```c
char c[8];
strcpy(c, "cd");
```

## 2.string method

### strlen, strcmp, strncmp, strcpy

```c string.h
int exile_strlen(const char *s) {
  int n;
  for (n = 0; *s != '\0'; n++, s++);
  return n;
}

int exile_strcmp(const char *a, const char *b) {
  while(*a++ == *b++)
    if (*(a-1) =='\0') return 0;

  /* while (*a == *b) {
   * a++;
   * b++;
   * if (*(b-1) == 0) {
   *   return 0;
   * }
   *}
   */
  if (*(a-1) > *(b-1)) return 1;
  return -1;
}

int exile_strncmp(const char *a, const char *b, int n) {
  int i = 0;
  while (i < n, *a++ == *b++) i++;
  //for (i = 0; i < n, *a++ ==*b++; i++);
  if(i == n) return 0;
  return *(a-1) - *(b-1);
}

void exile_strcpy(char *dest, const char *src) {
  while(*dest++ = *src++);
}

```

### find some word in a file

```c string_find.c
#include <stdio.h>
#include <stdlib.h>
#include "string.h"
#define N 10000

int main(int argc, char *argv[]) {
  if (argc != 3) {
    printf("%-8s%s\n%s\n%s\n",
      "Usage: ", "need two arguements",
      "1.string -- the word you want to find",
      "2.file -- the file you find the word in");
    exit(1);
  }

  int i, j, t, count = 0;
  char a[N], *p = argv[1];
  FILE *fin = fopen(argv[2], "r");
  if (!fin) {
    printf("%s: No such file\n", argv[2]);
    exit(1);
  }

  for (i = 0; i < N-1, (t = getc(fin)) != EOF; a[i] = t, i++);
  a[i] = '\0';

  for (i = 0; a[i] != '\0'; i++) {
    for (j = 0; p[j] != '\0'; j++) {
      if (a[i+j] != p[j]) break;
    }
    if (p[j] == '\0') {
      count++;
      printf("%d ", i);
    }
  }

  printf("\nuse strncmp: ");
  for (i = 0; i < exile_strlen(a); i++)
    if (exile_strncmp(&a[i], p, exile_strlen(p)) == 0)
      printf("%d " ,i);

  printf("\nfind %d \"%s\" in file %s\n", count, argv[1], argv[2]);
}

```
