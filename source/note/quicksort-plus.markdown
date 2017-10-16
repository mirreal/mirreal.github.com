---
layout: page
title: "quicksort_plus"
date: 2014-7-25 22:41
comments: true
sharing: true
footer: true
---

#### July 25th, 2014, Friday

## quicksort improved(20%-25%)

```c
void quicksort(Item a[], int l, int r) {
  if (r <= l) return;
  int i, mid = l + (r-l)/2;
  exch(a[mid], a[l+1]);
  compexch(a[r], a[l]);
  compexch(a[r], a[l+1]);
  compexch(a[l+1], a[l]);
  i = partition(a, l+1, r-1);
  quicksort(a, l, i-1);
  quicksort(a, i+1, r);
}
```

### Js

```javascript qsort.js
var sort = require('./sort.js');
var coch = sort.compexch;

function quicksort(a, l, r) {
  if (r-l <= 10) return;
  var mid = l + parseInt((r-l)/2);
  var t = a[mid];
  a[mid] = a[l+1];
  a[l+1] = t;
  coch(a, r, l);
  coch(a, r, l+1);
  coch(a, l+1, l);
  var i = partition(a, l+1, r-1);
  quicksort(a, l, i-1);
  quicksort(a, i+1, r);
}

function partition(a, l, r) {
  var i = l, j = r+1;
  var v = a[l];
  while (i < j) {
    while (a[++i] < v);
    while (v < a[--j]);
    if (i < j) {
      var t = a[i];
      a[i] = a[j];
      a[j] = t;
    }
  }
  var t = a[j];
  a[j] = a[l];
  a[l] = t;
  return j;
}

function insertion(a, l, r) {
  console.log('Insertion sorting...');
  for (var i = l; i <= r-1; i++) {
    for (var j = i+1, v = a[j]; j >= l+1 && v < a[j-1]; j -= 1) {
      a[j] = a[j-1];
    }
    a[j] = v;
  }
}

function qsort(a, l, r) {
  quicksort(a, l, r);
  insertion(a, l, r);
}

var N = 160000;
var a = [];
for (var i = 0; i < N; i++) {
  a[i] = parseInt(Math.random() * 1000000);
}
//console.log(a);
var start = Date.now();
/**
 * a.sort(function (x, y) {
 *   if (x < y) return -1;
 *   if (x > y) return 1;
 *   return 0;
 * });
*/
qsort(a, 0, N-1);
var end = Date.now();
time = end - start;
//console.log(a);
console.log('Time: ' + time + 'ms');
sort.check(a, 0, N-1);
```

### sort.js

```javascript sort.js
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

exports.compexch = function(a, i, j) {
  if (a[i] < a[j]) {
    var t = a[i];
    a[i] = a[j];
    a[j] = t;
  }
}
```
