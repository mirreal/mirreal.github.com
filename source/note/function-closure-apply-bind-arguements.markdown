---
layout: page
title: "Something basal about function"
date: 2014-7-12 10:24
comments: true
sharing: true
footer: true
---


## Closure

```javascript
function createFunctions() {
  var ans = [];

  for (var i = 0; i < 7; i++) {
    ans[i] = function() {
      return 4*i;
    };
  }
  return ans;
}
createFunctions().map(function (item) {
  console.log(item());
});
```

print:

```
    28
    28
    28
    28
    28
    28
    28
```

## call(), apply(), bind()

```javascript
var table = {
  entry: [{key: 1, value: 4}, {key: 'r', value: 't'}],
  addEntry: function(key, value) {
    this.entry.push({
      key: key,
      vlaue: value
    });
  },
  forEach: function(f, thisArg) {
    var entry = this.entry;

    for (var i = 0; i < entry.length; i++) {
      var ent = entry[i];
      f.call(thisArg, ent.key, ent.value);
    }
  }
};

var table2 = {entry: []};
table.forEach(table.addEntry, table2);

console.log(table2);
```

copy table.entry to table2

### object arguments

```javascript
function sum(a, b) {
  return a+b;
}

function sumApply(a, b) {
  return sum.apply(this, arguments);
}
```
