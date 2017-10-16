---
layout: page
title: "shaker"
date: 2014-7-23 22:29
comments: true
sharing: true
footer: true
---

#### July 23rd, 2014, Wedsday

## Shaker

```javascript
function shaker(a) {
  console.log('Shaker sorting...');
  var i = 0, m = a.length-1;
  var flag = 1;
  while(i < m) {
    if (flag == 1) {
      for (var j = m; j > i; j--) {
        compexch(a, j, j-1);
      }
      flag = 0;
      i++;
    } else {
      for (var n = i; n < m; n++) {
        compexch(a, n+1, n);
      }
      flag = 1;
      m--;
    }
  }
}

function compexch(a, i, j) {
  if (a[i] < a[j]) {
    var t = a[i];
    a[i] = a[j];
    a[j] = t;
  }
}
```

## 函数的传值

所有参数都是按值传递，简单类型传的是简单类型的副本，引用类型传递引用类型的副本。在函数改变其形参的引用，原本的值不会改变。
