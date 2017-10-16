---
layout: page
title: "qsort-3-and-select"
date: 2014-7-26 22:49
comments: true
sharing: true
footer: true
---

#### July 26th, 2014, Saturday

## quicksort three partition

### C

```c
void qsort_3part(Item a[], int l, int r) {
  if (r <= l) return;
  int i = l, j = r+1, p = l, q = r+1, k;
  Item v = a[l];

  while (i < j) {
    while (a[++i] < v);
    while (v < a[--j]);
    if (i < j) {
      exch(a[i], a[j]);
      if (a[i] == v) {
        p++;
        exch(a[p], a[i]);
      }
      if (a[j] == v) {
        q--;
        exch(a[q], a[j]);
      }
    }
  }
  exch(a[j], a[l]);
  i = j-1;
  j = j+1;
  for (k = l+1; k <= p; k++, i--) exch(a[k], a[i]);
  for (k = r; k >= q; k--, j++) exch(a[k], a[j]);
  qsort_3part(a, l, i);
  qsort_3part(a, j, r);
}
```

### JS

```javascript
function qsort_3part(a, l, r) {
  if (r <= l) return;
  var i = l, j = r+1, p = l, q = r+1;
  var v = a[l];

  while (i < j) {
    while (a[++i] < v);
    while (v < a[--j]);
    if (i < j) {
      exch(a, i, j);
      if (a[i] === v) {
        p++;
        exch(a, p, i);
      }
      if (a[j] === v) {
        q--;
        exch(a, q, j);
      }
    }
  }
  exch(a, j, l);
  i = j-1;
  j = j+1;
  for (var k = l+1; k <= p; k++, i--) exch(a, k, i);
  for (var k = r; k >= q; k--, j++) exch(a, k, j);
  qsort_3part(a, l, i);
  qsort_3part(a, j, r);
}
```

## Select

```c
void select_kth(Item a[], int l, int r, int k) {
  if (r <= l) return;
  int i = partition(a, l, r);
  if (i > k) select_kth(a, l, i-1, k);
  if (i < k) select_kth(a, i+1, r, k);
}
```

#### no recursion version

```c
void select_norec(Item a[], int l, int r, int k) {
  int i;
  while (r > l) {
    i = partition(a, l, r);
    if (i >= k) r = i-1;
    if (i <= k) l = i+1;
  }
}
```

#### check function

```c
void check_select(Item a[], int l, int r, int k) {
  Item v = a[k];
  int f = 1, i;
  for (i = l; i < k; i++) {
    if (a[i] > v) {
      f = -1;
      break;
    }
  }
  for (i = r; i > k; i--) {
    if (a[i] < v) {
      f = -1;
      break;
    }
  }
  if (f == 1) printf("Correct.\n");
  else printf("something is wrong. index: %d\n", i);
}
```
