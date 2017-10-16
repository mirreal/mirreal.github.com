---
layout: page
title: "quicksort"
date: 2014-7-24 22:34
comments: true
sharing: true
footer: true
---

#### July 24th, 2014, Thrusday

## Quicksort

### C

```c
int partition(Item a[], int l, int r) {
  int i = l-1, j = r;
  Item v = a[r];

  while(i < j) {
    while (a[++i] < v);
    while (v < a[--j]);
    if (i < j) exch(a[i], a[j]);
  }
  exch(a[i], a[r]);
  return i;
}

void quicksort(Item a[], int l, int r) {
  int i;
  if (r <= l) return;
  i = partition(a, l, r);
  quicksort(a, l, i-1);
  quicksort(a, i+1, r);
}
```

### Js

```javascript
function quicksort(a, l, r) {
  if (l >= r) return;
  var i = partition(a, l, r);
  quicksort(a, l, i-1);
  quicksort(a, i+1, r);
}

function partition(a, l, r) {
  var i = l-1, j = r;
  var v = a[r];
  while (i < j) {
    while (a[++i] < v);
    while (v < a[--j]);
    if (i < j) {
      var t = a[i];
      a[i] = a[j];
      a[j] = t;
    }
  }
  var t = a[i];
  a[i] = a[r];
  a[r] = t;
  return i;
}
```

#### sort check

```javascript
exports.check = function(a, l, r) {
  var ck = 1;
  for (var i = l; i < r; i++) {
    if (a[i] > a[i+1]) {
      ck = -1;
      break;
    }
  }
  if (ck == 1) {
    console.log('Sorting correct.');
  } else {
    console.log('Something is wrong. (index: ' + i + ')');
  }
}
```
