---
title: Nginx启用缓存功能
catalog: true
date: 2018-03-08 21:26:17
subtitle: 配置教程
header-img: "/img/article_header/article_header.png"
tags:
- Nginx
---

<font color=#bbb>内容来自摘录，如有侵权请及时联系我。</font>

## Nginx启用缓存功能

通过网络获取资源既速度缓慢又代价高昂：下载过程可能需要在客户端和服务器之间进行多次往返，这会导致延迟处理，并可能会阻止网页内容呈现，还会致使访问者支付数据费用。所有的服务器响应都应指定一种缓存政策，以帮助客户端确定是否以及何时能够重用之前获取的响应。


### 优化缓存的思路

1.html不做缓存，这样可以保证html每次请求都是最新的代码

2.js、css、图片等资源永久缓存，只需要从服务器上获取一次即可永久缓存到本地

3.js、css、图片要及时更新，保证这些文件有过修改后，能够快速更新

### Nginx设置


```shell
location ~*.*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm)$
{
  expires max;
}
 
 location ~ .*\.(?:js|css)$
{
  expires max;
}
 
location ~ .*\.(?:htm|html)$
{
  add_header Cache-Control "private, no-store, no-cache, must-revalidate, proxy-revalidate";
}
 
```
{% asset_img expires.png This is an nginx expires image %}

### 验证效果

更新前：
{% asset_img before.png This is a before response image %}

更新后：
{% asset_img after.png This is an after response image %}
