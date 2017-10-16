---
layout: page
title: "sorting-in-js"
date: 2014-7-22 22:20
comments: true
sharing: true
footer: true
---

#### July 22nd, 2014, Tuesday

### SOME WORDS

编码的世界里总是会遇到一些想象不到的Bug，当没有触发某个条件，它依然运转良好，甚至会认为完美至极，可是有一天达到某个值，于是大厦崩塌了。


----------

有时候稀里糊涂地东改西改，问题竟然像是解决了，我们也这样认为的时候，就是大错将至的时刻。

### Insertion sorting

```js
while (j >= l+1 && v < a[j-1])
```

## SORTING in JS

### Test

```javascript
var N = 16000;
var a = [];
for (var i = 0; i < N; i++) {
  a[i] = parseInt(Math.random() * 1000000);
}
```

### Bubble -- 800ms

```javascript
function bubble(a) {
  console.log('Bubble sorting...');
  for (var i = 0; i < a.length; i++) {
    for (var j = a.length-1; j > i; j--) {
      if (a[j] < a[j-1]) {
        var t = a[j];
        a[j] = a[j-1];
        a[j-1] = t;
      }
    }
  }
}
```

### Selection  -- 500ms

```javascript
function selection(a) {
  console.log("Selection sorting...");
  for (var i = 0; i < a.length; i++) {
    var min = i;
    for (var j = i+1; j < a.length; j++) {
      if (a[j] < a[min]) min = j;
    }
    var t = a[min];
    a[min] = a[i];
    a[i] = t;
  }
}
```

### Insertion

-- 600ms

```javascript
function insertion_dir(a) {
  console.log('Direct insertion sorting...');
  for (var i = 0; i < a.length-1; i++) {
    for (var j = i+1; j >= 1; j--) {
      if (a[j] < a[j-1]) {
        var t = a[j];
        a[j] = a[j-1];
        a[j-1] = t;
      }
    }
  }
}
```

--250ms

```javascript
function insertion(a) {
  console.log('Insertion sorting...');
  for (var i = 0; i < a.length-1; i++) {
    for (var j = i+1, v = a[j]; j >= 1 && v < a[j-1]; j--) {
      a[j] = a[j-1];
    }
    a[j] = v;
  }
}
```

### Shell  -- 10ms

```javascript
function shell(a) {
  console.log("Shell sorting...");
  for (var h = 1; h < a.length/3; h = 3*h+1);
  while(h >= 1) {
    for (var i = 0; i < a.length-h; i++) {
      for (var j = i+h, v = a[j]; j >= h && v < a[j-h]; j = j-h) {
        a[j] = a[j-h];
      }
      a[j] = v;
    }
    h = (h-1)/3;
  }
}
```

#### 小小的改动就可能就有很大的效率提升
