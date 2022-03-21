---
title: Canvas的基本用法
catalog: true
date: 2022-03-21 23:33:19
subtitle: Canvas系列
header-img: '/img/article_header/article_header.png'
tags: Canvas
---

# Canvas 教程

`<canvas>`是一个可以使用`JavaScript`脚本来绘制图形的`HTML`元素。例如，它可以用于绘制图表、制作图片构图或者制作简单的动画。

`<canvas>`最早由 Apple 引入`Webkit`，用于 Mac OS X 的 Dashboard，随后被各个浏览器实现。如今，所有主流的浏览器都支持它。Canvas 的默认大小为 300 像素 ×150 像素（宽 × 高，像素的单位是 px）。但是，可以使用 HTML 的高度和宽度属性来定义 Canvas 的尺寸。为了在 Canvas 上绘制图形，我们使用一个 Javascript 上下文对象，它能动态创建图像（creates graphics on the fly）。

## 基础用法

### `<canvas>`元素

```js
<canvas id="tutorial" width="150" height="150"></canvas>
```

1. `canvas`和`img`元素很相像，不同是它并没有`src`和`alt`属性，且`</canvas>`标签不可省略，如果不存在，则文档的其余部分会被认为是替代内容，将不会显示出来。。
2. `canvas`标签只有两个属性：`width`和`height`。可通过 DOM 实例修改。默认值为：300 像素\*150 像素。
3. 可以使用`CSS`来定义大小（不推荐使用），但在绘制时图像会伸缩以适应它的框架尺寸：如果`CSS`的尺寸与初始画布的比例不一致，它会出现扭曲。
4. `margin` `border` `background`等属性，不会影响`canvas`中的实际图像。默认`canvas`状态为完全透明。

## 替换内容

对于较老的浏览器（尤其是 IE9 之前的 IE 浏览器）可以通过设置替换内容。用于优雅降级兼容处理。举个栗子 🌰，我们可以提供对`canvas`内容的文字描述或者是提供动态生成内容相对应的静态图片，如下所示：

```html
<canvas id="stockGraph" width="150" height="150">
  current stock price: $3.15 +0.15
</canvas>

<canvas id="clock" width="150" height="150">
  <img src="images/clock.png" width="150" height="150" alt="" />
</canvas>
```

## 渲染上下文（the rendering context）

`canvas`元素上有一个`getContext()`的方法，这个方法是用来获得渲染上下文和它的绘画功能。方法接受一个参数，即上下文的类型。对于 2D 图像而言，可以使用`CanvasRenderingContext2D`。

```js
var canvas = document.getElementById('tutorial');
var ctx = canvas.getContext('2d'); // '2d'作为参数获取到的即为：CanvasRenderingContext2D 实例
```

有了上下文，您就可以绘制任何想要的东西。

## 检查支持性

替换内容是用于在不支持`canvas`标签的浏览器中展示的。通过简单的测试`getContext()方法的存在，脚本可以检查编程支持性`。上面的代码片段现在变成了这个样子：

```js
var canvas = document.getElementById('tutorial');

if (canvas.getContext) {
  var ctx = canvas.getContext('2d');
  // drawing code here
} else {
  // canvas-unsupported code here
}
```

## 一个简单例子

下面我们来看个简单的例子，绘制了两个长方形，其中的一个有着 alpha 透明度。我们将在接下来的内容里深入探索一下这是如何工作的。

```html
<html>
  <head>
    <script type="application/javascript">
      function draw() {
        var canvas = document.getElementById('canvas');
        if (canvas.getContext) {
          var ctx = canvas.getContext('2d');

          ctx.fillStyle = 'rgb(200,0,0)';
          ctx.fillRect(10, 10, 55, 50);

          ctx.fillStyle = 'rgba(0, 0, 200, 0.5)';
          ctx.fillRect(30, 30, 55, 50);
        }
      }
    </script>
  </head>
  <body onload="draw();">
    <canvas id="canvas" width="150" height="150"></canvas>
  </body>
</html>
```

效果如下：
{% asset_img canvas_ex1.png This is an canvas example image %}
