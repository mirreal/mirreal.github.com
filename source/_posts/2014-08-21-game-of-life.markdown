---
layout: post
title: "康威生命游戏的简单实现"
date: 2014-08-21 19:26:12 +0800
comments: true
categories:
---

生命游戏，数学家John Conway发明的一个游戏，又称康威生命演化，生命棋，细胞自动机。

康威有许多好玩有趣的发明，最广为人知的一个是外观数列（Look-and-Say），这里不多说，另一个就是生命游戏（Game-of-Life）。

关于康威，摘录一段Wikipedia的叙述:
{% blockquote 約翰·何頓·康威, wikipedia%}
約翰·何頓·康威（John Horton Conway，1937年12月26日－），生於英國利物浦，數學家，活躍於有限群的研究、趣味數學、紐結理論、數論、組合博弈論和編碼學等範疇。

康威年少時就對數學很有強烈的興趣：四歲時，其母發現他背誦二的次方；十一歲時，升讀中學的面試，被問及他成長後想幹甚麼，他回答想在劍橋當數學家。後來康威果然於劍橋大學修讀數學，現時為普林斯頓大學的教授。
{% endblockquote %}

生命游戏模拟的是二维平面上生命的演化过程。

<!-- more -->

规则很简单：每个细胞有两种状态--存活或死亡，每个细胞与以自身为中心的周围八格细胞产生互动。

1. 如果一个活细胞周围有2至3个活细胞，在下一个阶段继续存活，否则死亡；
2. 如果一个死细胞周围有3个活细胞，在下一个阶段将变成活细胞，否则继续保持死亡

康威生命游戏是简单规则产生复杂变化的典型例子。在演变过程中，可以看到一些非常美妙的变化，和一些优美的几何图形。


#### 下面是用HTML5 Canvas实现的一个简单版本。

源码：[https://github.com/mirreal/moon9/tree/master/GameOfLife](https://github.com/mirreal/moon9/tree/master/GameOfLife)

DEMO：[http://mirreal.net/game-of-life/](http://mirreal.net/game-of-life/)

JS代码如下：

```javascript life.js
function Game() {
  this.stones = [];
  this.canvas = new Canvas();

  this.init();
}

Game.prototype.init = function() {
  var self = this;

  this.createRandomStones();
  this.draw();

  this.getAroundStones();

  this.eventHandler();

  this.loop = setInterval(function() {
    self.update();
    self.draw();
  }, 120);
};

Game.prototype.eventHandler = function() {
  var self = this;

  var snapshotButton = document.getElementById('snapshotButton'),
      snapshotImageElement = document.getElementById('snapshotImageElement'),
      canvas = document.getElementById('canvas');

  snapshotButton.onclick = function(event) {
    event.preventDefault();

    if (snapshotButton.innerHTML == 'Snapshot') {
      clearInterval(self.loop);
      var dataUrl = canvas.toDataURL();
      snapshotImageElement.src = dataUrl;
      snapshotImageElement.style.display = 'inline';
      canvas.style.display = 'none';
      snapshotButton.innerHTML = 'Continue';
    } else {
      self.loop = setInterval(function() {
        self.update();
        self.draw();
      }, 120);
      canvas.style.display = 'inline';
      snapshotImageElement.style.display = 'none';
      snapshotButton.innerHTML = 'Snapshot';
    }
  };
};

Game.prototype.createRandomStones = function() {
  for (var i = 0; i < 32; i++) {
    for (var j = 0; j < 32; j++) {
      var status = Math.random() < 0.2 ? true : false;
      this.stones.push(new Stone({x: i, y: j}, status))
    }
  }
};

Game.prototype.draw = function() {
  var self = this;

  this.canvas.drawGrid("lightgrey", 20, 20);
  this.stones.forEach(function(stone) {
    if (stone.status === true) {
      self.canvas.drawStone(stone);
    }
  });
};

Game.prototype.getAroundStones = function() {
  var self = this;

  this.stones.forEach(function(stone) {
    stone.around.forEach(function(position) {
      stone.aroundStones.push(self.stones[32*position.x + position.y]);
    });
  });
};

Game.prototype.update = function() {
  var self = this;

  this.stones.forEach(function(stone) {
    stone.aroundStones.forEach(function(s) {
      if (s.status === true) stone.aliveCount += 1;
    });

    if (stone.status === true) {
      if (stone.aliveCount === 2 || stone.aliveCount === 3) {
        stone.nextStatus = true;
      } else {
        stone.nextStatus = false;
      }
    } else {
      if (stone.aliveCount === 3) stone.nextStatus = true;
      else stone.nextStatus = false;
    }
  });

  this.stones.forEach(function(stone) {
    stone.status = stone.nextStatus;
    stone.aliveCount = 0;
  });
}




function Stone(position, status) {
  this.x = position.x;
  this.y = position.y;

  this.status = status;
  this.nextStatus = false;

  this.aroundStones = [];
  this.aliveCount = 0;

  this.around = [];
  this.getAround();
}

Stone.prototype.getAround = function() {
  for (var i = this.x-1; i <= this.x+1; i++) {
    for (var j = this.y-1; j <= this.y+1; j++) {
      if (i == this.x && j == this.y) continue;
      if (i < 0 || i >= 32) continue;
      if (j < 0 || j >= 32) continue;
      this.around.push({x: i, y: j});
    }
  }
};


function Canvas() {
  this.canvas = document.getElementById('canvas');
  this.context = canvas.getContext('2d');
}

Canvas.prototype.drawGrid = function(color, stepx, stepy) {
  var canvas = this.canvas;
  var ctx = this.context;

  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.save();
  ctx.strokeStyle = color;
  ctx.lineWidth = 0.5;

  for (var i = stepx + 0.5; i < ctx.canvas.width; i += stepx) {
    ctx.beginPath();
    ctx.moveTo(i, 0);
    ctx.lineTo(i, ctx.canvas.height);
    ctx.stroke();
    ctx.closePath();
  }

  for (var i = stepy + 0.5; i < ctx.canvas.height; i += stepy) {
    ctx.beginPath();
    ctx.moveTo(0, i);
    ctx.lineTo(ctx.canvas.width, i);
    ctx.stroke();
    ctx.closePath();
  }
  ctx.restore();
};

Canvas.prototype.drawStone = function(stone) {
  var ctx = this.context;

  var x = 20 * stone.x + 10;
  var y = 20 * stone.y + 10;

  ctx.fillStyle = 'orange';
  ctx.beginPath();
  ctx.arc(x, y, 9, 0, Math.PI*2, false);
  ctx.closePath();
  ctx.fill();
};


new Game();

```


#### TIPS:

在Google搜索 `Conway's Game of Life` ,会看到Google的一个实现。
