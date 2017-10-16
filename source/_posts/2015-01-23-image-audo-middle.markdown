---
layout: post
title: "实现不定尺寸图片居中的几种方式"
date: 2015-01-23 21:26:48 +0800
comments: true
categories:
---

这是在实习工作中遇到的一个需求，实现未知尺寸的图片在一固定宽高的容器中水平和垂直居中。


比如是在一个 200px 的正方形容器，图片的宽高可能大于 200px，也可能小于 200px。对于小图片，就是简单的水平和垂直居中，而大图片则要求按比例缩放，并实现居中。

这个问题主要有两个方面的点：

* 未知尺寸元素的垂直居中
* 大图片的宽高适配


因为图片大小尺寸未知，且可能超出容器，很多垂直居中的方法不能用。

### 最终的解决方法如下：

<!-- more -->

HTML 大概是这样的：

```html
  <div class="box">
    <img src="img/0.jpg" alt="earth shadow" title="It's me">
  </div>
```

CSS：

```css
.box {
  width: 200px;
  height: 200px;
  position: relative;
  text-align: center;
}
.box img {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  max-width: 200px;
  max-height: 200px;
}
```

将容器的位置设为 `relative` ，图片的位置设为 `absolute` ，然后将其 `top`， `right`， `bottom`， `left` 都设置成0，虽然图片尺寸未知，但是对于一张具体的图片而言是有固定宽高的，所以并不能使它四个方位间距都为0，因此 `margin: auto;` 会让它居中。如果仅对于宽高都小于200px的图片，这样就足够了，但是对于大图片就不适用，所以再加上 `max-width: 200px; max-height: 200px;` ，这样图片就可以自适应缩放。


查看演示：[demo1](https://mirreal.github.io/demo/image-center/1.html)

### 方式二：使用表格布局

还是这段 HTML：

```html
<div class="box1">
  <img src="img/0.jpg"  alt="earth shadow" title="It's me">
</div>
```

CSS这样写：

```css
.box1 {
  width: 200px;
  height: 200px;
  min-width: 200px;
  display: table-cell;
  vertical-align: middle;
  text-align: center;
}
.box1 img {
  max-width: 200px;
  max-height: 200px;
  vertical-align: middle;
}
```

查看演示：[demo2](https://mirreal.github.io/demo/image-center/2.html)

### 方式三：使用背景图片

HTML就直接写了：

```html
<div class="box2"></div>
```

CSS：

```css

.box2 {
  width: 200px;
  height: 200px;
  background-image: url(img/0.jpg);
  background-repeat: no-repeat;
  background-position: center;
}
```

一个演示：[demo3](https://mirreal.github.io/demo/image-center/3.html)

直接使用 `background-position: center;` 实现居中，如图片尺寸小于 200px 就 OK 了。但是对于大图片，会被截掉，所以还需加上 `background-size: contain;` ，可是对于小图片又有问题，因为加上这个属性会对图片缩放，小图也不例外。而在我们的需求里，对小图要求按照其原有大小显示。

貌似这种方式使用纯 CSS 行不通。

...
