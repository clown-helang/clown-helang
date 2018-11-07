---
title: Nginx启用缓存功能
catalog: true
date: 2018-03-03 19:12:34
subtitle: 配置教程
header-img: "/img/article_header/article_header.png"
tags:
- Nginx
---

<font color=#bbb>内容来自摘录，如有侵权请及时联系我。</font>

## Nginx启用压缩功能

所有现代浏览器都支持 gzip 压缩并会为所有 HTTP 请求自动协商此类压缩。启用 gzip 压缩可大幅缩减所传输的响应的大小（最多可缩减 90%），从而显著缩短下载相应资源所需的时间、减少客户端的流量消耗并加快网页的首次呈现速度。


### 启用方法

修改nginx配置nginx.conf


```

# gzip
gzip on;
gzip_buffers 32 4k;
gzip_comp_level 6;
gzip_min_length 200;
gzip_types text/css text/xml application/javascript;
gzip_vary on;

```

{% asset_img config.png This is an nginx config image %}

修改完成后，重启nginx即可生效。


参数 | 解释说明
---|---
gzip on | 开启gzip压缩功能
gzip_buffers 32 4k | 或者 gzip_buffers 16 8k;  //这里表不每压缩32个包，每个包4k大小，就向客户端发送
gzip_comp_level 6 | 这里表示压缩级别，可以是0到9中的任一个，级别越高，压缩就越小，节省了带宽资源，但同时也消耗CPU资源，所以一般折中为6
gzip_min_length 200 | 这里表示如果文件小于200个字节，就不用压缩，因为没有意义，本来就很小
gzip_types text/css text/xml application/javascript | 这里表示哪些类型的文件要压缩，text/html类型是默认的不需要写，如果不知道文件有哪些类型，可以在nginx目录中找到文件类型，/var/mywww/nginx/conf/mime.types 文件中记录了所有可以 压缩的文件类型
gzip_vary on | 可以不写，表示我在传送数据时，给客户端说明我使用了gzip压缩


### 验证效果 
通过http工具，查看任意文件的http信息,Response Header 出现Content-Encoding:gzip，即表示启用成功。

{% asset_img response.png This is an http response image %}
