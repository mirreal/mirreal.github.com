---
layout: post
title: "三种变长编码方法的C实现"
date: 2014-02-14 10:27:39 +0800
comments: true
categories:
---


信息编码是数据压缩的的基础理论。常用的变长编码法有三种：香农（Shannon）编码，费诺（Fano）编码，霍夫曼（Huffman）编码。

通常情况下，霍夫曼编码法的编码效率最优。

## 1.香农编码法

香农编码法是一种很基础的编码方法，效率很低。

方法如下：

1. 将M个信源按其概率递减顺序排列
2. 计算各个消息的计算码字长度：`-log q(Xm)`向上取整
3. 计算地 m 个消息的累加概率并转换成二进制
4. 取小数点后第 m 个消息码字长度位即为该消息码字


香农编码法的C语言实现：

<!-- more -->

```c shannon.c
// 编译指令：gcc shannon.c -lm

#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main() {
  int i, j, n;

  printf("请输入信源符号个数：");
  scanf("%d", &n);
  printf("请输入各符号的概率：");

  double x[n];
  for (i = 0; i < n; i++) {
    printf("x[%d]= ", i);
    scanf("%lf", &x[i]);
  }

// 选择排序（降序）
  for (i = 0; i < n-1; i++) {
    double v;
    for (j = i+1, v = x[j]; v > x[j-1] && j >= 1; j--) {
      x[j] = x[j-1];
    }
    x[j] = v;
  }

// 计算码长：1-log2(p(xi))向上取整
  int k[n];
  for (i = 0; i < n; i++) {
    k[i] = -log(x[i]) / log(2) + 1;
    if (k[i] == (-log(x[i]) / log(2) + 1)) k[i] -= 1;
  }

// 累加概率
  double pa[n];
  pa[0] = 0.0;
  for (i = 1; i < n; i++) {
    pa[i] = pa[i-1] + x[i-1];
  }

// 将累加概率转换为二进制
  char code[n][n];
  for (i = 0; i < n; i++) {
    double t = pa[i];
    for (j = 0; j < k[i]; j++) {
      double temp = 2*t;
      if (temp > 1) {
        code[i][j] = '1';
        t = 2*t - 1;
      } else {
        code[i][j] = '0';
        t = 2*t;
      }
    }
  }

// 输出结果
  printf("%16s %12s %16s %4s %4c%s\n", "信源", "概率p(x)", "累加概率", "码长", ' ', "码字code");
  for (i = 0; i < n; i++) {
    printf("%12d %12lf %12lf %4d %4c", i+1, x[i], pa[i], k[i], ' ');
    for (j = 0; j < k[i]; j++) {
      printf("%c", code[i][j]);
    }
    printf("\n");
  }
  return 0;
}
```


## 2.费诺编码法

这个编码方法准确说应该叫做 Shannon-Fano 编码法。这项技术是香农于1948年，在他介绍信息理论的文章“通信数学理论”中被提出的，归功于费诺，是由于他在不久以后以技术报告发布了它。

方法如下：

1. 将M个信源按其概率递减顺序排列
2. 对消息集按概率大小分解成两个子集，使两子集概率之和尽可能相等
3. 将第一个子集编码为“0，第二个编码为”1“
4. 对子集进行递归操作，知道子集仅含一个消息
5. 将逐次分解过程中的码元排列起来即为各消息码字

香农-费诺编码法的C语言实现：

```c fano.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

int n;
double* x;
char **code;

void fano(int a, int b) {
  if (b-a < 1) return;
  int i;
  double sum = 0;
  for (i = a; i <= b; i++) {
    sum += x[i];
  }
  double s[n], pa = 0;
  for (i = a; i <= b; i++) {
    pa += x[i];
    s[i] = fabs(2*pa - sum);
  }
  int min = a;
  for (i = a+1; i <= b; i++) {
    if(s[i] <= s[min]) min = i;
  }
  for (i = a; i <= b; i++) {
    if (i <= min) strcat(code[i], "0");
    else strcat(code[i], "1");
  }
  fano(a, min);
  fano(min+1, b);
}

int main() {
  int i, j, n;
  printf("请输入信源符号个数：");
  scanf("%d", &n);
  printf("请输入各符号的概率：");

  x = malloc(n * sizeof(double));
  for (i = 0; i < n; i++) {
    printf("x[%d]= ", i);
    scanf("%lf", &x[i]);
  }

  for (i = 0; i < n-1; i++) {
    double v;
    for (j = i+1, v = x[j]; v > x[j-1] && j >= 1; j--) {
      x[j] = x[j-1];
    }
    x[j] = v;
    //printf("x[%d]= %lf", i, x[i]);
  }

  code = malloc(n * sizeof(char*));;
  for (i = 0; i < n; i++) {
    code[i] = malloc(n * sizeof(char));
  }
  fano(0, n-1);
  printf("%16s %12s %4c%s\n", "信源", "概率p(x)", ' ', "码字code");
  for (i = 0; i < n; i++) {
    printf("%12d %12lf %4c %s\n", i+1, x[i], ' ', code[i]);
  }
  return 0;
}
```

## 3.霍夫曼编码法

香农当然知道 Shannon-Fano 编码法不是最优的，果然没太久，费诺的学生霍夫曼就找到一种更优的编码方法。

有过算法课程上霍夫曼树的经验，霍夫曼编码法就比较容易理解。每次选取最小的节点构造霍夫曼树，各消息的码字即为从根节点到该消息节点的码元组合。

霍夫曼编码法的C语言实现：

```c huffman.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_VALUE 1

typedef char** huffman_code;
typedef struct {
  double weight;
  int parent, lchild, rchild;
} HTNode, *huffman_tree;
typedef struct {
  int s1;
  int s2;
} min_code;

huffman_code Huffman_coding(huffman_tree HT, huffman_code HC, double *x, int n);
min_code select_min(huffman_tree HT, int n);

int main() {
  huffman_tree HT = NULL;
  huffman_code HC = NULL;
  int i, n;
  printf("请输入信源符号个数：");
  scanf("%d", &n);
  double *x = malloc((n+1) * sizeof(double));
  x[0] = 0;
  printf("请输入各符号的概率：");
  for (i = 1; i <= n; i++) {
    printf("X[%d]= ", i);
    scanf("%lf", &x[i]);
  }

  HC = Huffman_coding(HT, HC, x, n);
  printf("\nhuffman_code:\n");
  printf("Number Weight Code\n");
  for (i = 1; i <= n; i++) printf("%-6d %-6lf %-4s\n", i, x[i], HC[i]);
}

huffman_code Huffman_coding(huffman_tree HT, huffman_code HC, double *x, int n) {
  int i, s1 = 0, s2 = 0;
  char *code;
  int f, c, start, m;
  huffman_tree p;
  min_code min;
  if (n <= 1) return;
  m = 2*n - 1;
  HT = malloc((m+1) * sizeof(HTNode));

  for (p = HT, i = 0; i <= n; i++, p++, x++) {
    p->weight = *x;
    p->parent = 0;
    p->lchild = 0;
    p->rchild = 0;
  }
  for (; i <= m; i++, p++) {
    p->weight = 0;
    p->parent = 0;
    p->lchild = 0;
    p->rchild = 0;
  }
  for (i = n+1; i <= m; i++) {
    min = select_min(HT, i-1);
    s1 = min.s1;
    s2 = min.s2;
    HT[s1].parent = i;
    HT[s2].parent = i;
    HT[i].lchild = s1;
    HT[i].rchild = s2;
    HT[i].weight = HT[s1].weight + HT[s2].weight;
  }

  printf("HT List:\n");
  printf("%8s %8s %8s %8s %8s\n", "Number", "weight", "parent", "lchild", "rchild");
  for (i = 1; i <= m; i++) {
    printf("%8d %8lf %8d %8d %8d\n",
      i, HT[i].weight, HT[i].parent, HT[i].lchild, HT[i].rchild);
  }
  HC = malloc((n+1) * sizeof(char *));
  code = malloc(n * sizeof(char *));
  code[n-1] = '\0';
  for (i = 1; i <= n; i++) {
    start = n-1;
    for (c = i, f = HT[i].parent; f != 0; c = f, f = HT[f].parent) {
      if (HT[f].lchild == c) code[--start] = '0';
      else code[--start] = '1';
    }
    HC[i] = malloc((n-start) * sizeof(char *));
    strcpy(HC[i], &code[start]);
  }
  free(code);
  return HC;
}

min_code select_min(huffman_tree HT, int n) {
  min_code code;
  int s1, s2;
  double m1, m2;
  int i;
  s1 = s2 = 1;
  m1 = m2 = MAX_VALUE;
  for (i = 1; i <= n; i++) {
    if (HT[i].parent == 0) {
      if (HT[i].weight < m1) {
        m2 = m1;
        s2 = s1;
        m1 = HT[i].weight;
        s1 = i;
      } else if (HT[i]. weight < m2) {
        m2 = HT[i].weight;
        s2 = i;
      }
    }
  }

  code.s1 = s1;
  code.s2 = s2;
  return code;
}
```
