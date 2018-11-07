---
title: Nginx配置代理解决跨域问题
catalog: true
date: 2018-3-23 13:56:11
subtitle: 配置教程
header-img: "/img/article_header/article_header.png"
tags:
- Nginx
---

<font color=#bbb>内容来自摘录，如有侵权请及时联系我。</font>

## Nginx配置代理解决跨域问题

目前流行通过cros解决跨域问题，但是这么做有一个前提，就是需要后端的配合，后台应用服务需要增加跨域的支持，需要修改的工作量以及影响范围不可预估，所以，我们可以通过ngnix转发的方式来变向的脱离对后台服务跨域设置的限制：

假设后台服务为： http://192.168.1.1:8080/webservices/getUser

前台部署的nginx服务器配置为：

```
http {
    gzip on;
    gzip_buffers 32 4k;
    gzip_comp_level 6;
    gzip_min_length 200;
    gzip_types text/css text/xml application/javascript;
    gzip_vary on;
    server
    {
        listen 80;
        server_name localhost;
        location /
        {
            root html;
            index index.html index.htm;
        }
    }
}
```
我们只要在里面加上转发即可：

```
http {
    gzip on;
    gzip_buffers 32 4k;
    gzip_comp_level 6;
    gzip_min_length 200;
    gzip_types text/css text/xml application/javascript;
    gzip_vary on;
    server
    {
        listen 80;
        server_name localhost;
        location /
        {
            root html;
            index index.html index.htm;
        }
        location /webservices{
            if ( $request_method = OPTIONS ) {
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
                return 200;
            }
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Access-Control-Allow-Headers, Authorization';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
            proxy_pass http://192.168.1.1:8080/webservices;
        }
    }
}
```

