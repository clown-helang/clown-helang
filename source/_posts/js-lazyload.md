---
title: 基于JavaScript节流和防抖动实现的懒加载效果
catalog: true
date: 2018-11-28 16:08:53
subtitle: 性能优化
header-img: "/img/article_header/article_header.png"
tags:
- Javascript
---

对于页面内图片资源丰富的网站来说，懒加载对于提升用户首屏加载速度是至关重要的优化。下面就来看看具体如何实现懒加载效果。

## 懒加载
单纯的图片懒加载思想比较简单，通过js获取img标签的集合，然后遍历这个img集合判断当前img是否位于可视窗口之内，如果在，则去加载真正的图片src，如果不在则不加载。下面通过代码看看具体实现方式：

```
<!--lazyload.html-->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Lazy-Load</title>
    <link rel="stylesheet" type="text/css" href="./css/lazyload.css"/>
</head>
<body>
    <div class="container">
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/1.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/2.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/3.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/4.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/5.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/6.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/7.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/8.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/9.jpg"/>
        </div>
        <div class="img">
            <img class="picture" alt="加载中" data-src="./images/10.jpg"/>
        </div>
    </div>
</body>
<script src="./js/lazyload.js"></script>
<script>
    window.onload = function () {
        lazyLoad();
        window.addEventListener('scroll', lazyLoad);
    }
</script>
</html>
```

```
//lazyload.js

//获取所有img标签,定义为全局变量避免后面反复获取dom节点影响性能。
var imgs = document.getElementsByTagName('img');
//获取当前窗口高度
var viewHeight = window.innerHeight || window.documentElement.clientHeight;
//用于记录当前显示到了第几张，避免每次都从第一张图片开始检查是否露出
var num = 0;

function lazyLoad() {
    for(var i = num; i < imgs.length; i++ ){
        //用可视区域高度减去当前元素顶部距离可视区域顶部高度
        var distance = viewHeight - imgs[i].getBoundingClientRect().top;
        //如果可视区域高度大于等于元素顶部距离可视区域顶部高度，说明元素露出
        if(distance >= 0){
            //给元素写入真实的src，展示图片
            imgs[i].src = imgs[i].getAttribute('data-src');
            //前i张图片已经加载完毕，下次从第i+1张开始检查是否露出
            num = i + 1
        }
    }
}

```

上面的例子就是一个实现懒加载的简单过程，但是这种方式还存在一些问题，如果用户在使用过程中反复的持续滚动页面，这样对于性能的消耗也是巨大的因为每一次的滚动事件都会被监听事件监听到并执行回调函数。所以需要对此进行相应的优化，下面来讲解一下如何通过节流（throttle）和防抖动（debounce）实现懒加载的性能优化。

## 节流(throttle)
节流的基本思想就是说我再短时间内如果高频率尝试触发同一个操作，那么我就设置一个间隔时间interval，如果上一次触发的时间与这一次的触发时间间隔已经大于等于当前时间时再去调用一次回调方法。下面看一下具体代码实现：

```
function throttle(fn, interval) {
    var last = 0;
    return function () {
        //保留调用时的this上下文
        var _this = this;
        //保留调用时传入的参数
        var args = arguments;
        /*
        * 记录本次触发回调的时间。
        * 不添加 “+” 返回的是 “Wed Nov 28 2018 13:59:55 GMT+0800 (中国标准时间)”
        * 添加“+” 返回的是 “1543384765179”
        * */
        var now = +new Date();
        if(now - last >= interval){
            last = now;
            fn.apply(_this, args)
        }
    }
}
```

网页中通过监听事件调用

```
<!--lazyload.html-->

window.addEventListener('scroll', throttle(()=>console.log("***触发了滚动事件***"),1000));
```

上面例子中delay为1000毫秒，所以如果用户在5000毫秒之内持续滚动页面。Throttle会触发5次 fn 滚动事件。从而可以有效的避免性能浪费。

但是这种方式也存在有一定的问题加入用户滚动的持续时间为5500毫秒，那么5000-5500毫秒这一段时间的滚动事件就会被忽略掉。所以单纯的节流方式也不是最佳的优化方案。

## 防抖动(debounce)

防抖动在与上面的实现方式上有一些不同，防抖动是通过设置定时器，传入一个延时(delay)时间，持续滚动过程不断重置定时器，知道滚动结束并且到达延时(delay)时间才会触发回调函数 fn，具体代码如下：

```
function debounce(fn, delay) {
    var timer = null;

    return function () {
        var _this = this;
        var args = arguments;
        if(timer){
            clearTimeout(timer)
        }
        timer = setTimeout(function () {
            fn.apply(_this, args)
        }, delay)
    }
}
```
网页中通过监听事件调用
```
<!--lazyload.html-->

window.addEventListener('scroll', debounce(()=>console.log("***触发了滚动事件***"),1000));
```
但是这种方式也存在一个问题，就是如果用户持续滚动页面，那页面就始终不会去执行懒加载的回调方法，这样同样导致页面会出现内容。所以需要与上面讲的节流方式合作使用才可以真正达到想要的效果。

## 节流&防抖动
基本思想是通过节流方式去避免防抖动方式存在长等待的问题。

```
/*
* thrAndDeb: throttle-and-debounce
* */
function thrAndDeb(fn, delay) {
    var last = 0, timer = null;
    return function () {
        //保留调用时的this上下文
        var _this = this;
        //保留调用时传入的参数
        var args = arguments;
        /*
        * now: 记录本次触发回调的时间。
        * 不添加 “+” 返回的是 “Wed Nov 28 2018 13:59:55 GMT+0800 (中国标准时间)”
        * 添加“+” 返回的是 “1543384765179”
        * */
        var now = +new Date();

        if(now - last < delay){
            clearTimeout(timer);
            timer = setTimeout(function () {
                last = now;
                fn.apply(_this, args)
            }, delay)
        } else {
            last = now;
            fn.apply(_this, args);
        }
    }
}

```
网页中通过监听事件调用
```
<!--lazyload.html-->

window.addEventListener('scroll', thrAndDeb(lazyLoad,1000));
```


同样，假设delay为1000毫秒，如果用户在5500毫秒之内持续滚动页面。使用thrAndDeb会触发6次滚动事件。前五次和Throttle触发时间是一样的，最后一次是由Debounce在第6500毫秒时相应第5500毫秒时的那一次滚动。这样就可以即减少了调用fn的频率，又防止了调用丢失。
