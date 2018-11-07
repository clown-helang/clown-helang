---
title: Centos下安装Nginx
catalog: true
toc_nav_num: true
date: 2018-02-24 21:01:06
subtitle: 安装教程
header-img: "/img/article_header/article_header.png"
tags:
- Nginx
---

# Centos下安装 Nginx

## nginx安装命令

```
# 建立nginx的yum仓库
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# 安装
# 由于CENTOS7.2默认使用老版本的openssl(OpenSSL 1.0.1e) ,这个问题会导致yum nginx-1.12以上版本的时候会因为依赖libcrypto.so.10(OPENSSL_1.0.2)(64bit)的问题造成安装失败。解决办法：1、从http://rpm.pbone.net 搜索相关的RPM包下载（openssl-libs-1.0.2k-8.el7.x86_64.rpm openssl-1.0.2k-8.el7.x86_64.rpm）；2、以上两个文件下载到本地，然后安装yum localinstall openssl-libs-1.0.2k-8.el7.x86_64.rpm openssl-1.0.2k-8.el7.x86_64.rpm。安装好之后，就可以正常安装nginx了。
sudo yum install nginx
# 查看版本
nginx -v
```

## 创建conf文件夹、logs文件夹

```
cd /usr/share/nginx
# 创建 logs 及 conf 文件夹
mkdir logs/ conf/
```

## 在 conf 文件夹下创建 nginx.conf 文件。


```
error_log /usr/share/nginx/logs/error.log;
 
events {
    # 每一个worker进程能并发处理（发起）的最大连接数，计算较为复杂，如果需要详细的配置规则请自行查询详细资料，默认可以配置大一些
    worker_connections  10240;
}
 
http {
    # 配置支持解析文件类型
    include    mime.types;
    # 配置允许跨域
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Headers X-Requested-With;
    add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
 
    #配置H5页面访问端口和根路径映射
    server {
        listen       80;
        # 注意此处写的是部署的服务器真实地址
        server_name  10.0.251.246;
 
        location / {
            # 以H5构建的网页放的位置为准，推荐直接放在nginx的html文件夹下
            root   /usr/share/nginx/html/StarfsaAppHTML;
            index  index.html index.htm;
        }
 
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 
    }
 
    #配置api访问转发
    server {
        listen       10080;
        # 注意此处写的是部署的服务器真实地址
        server_name  10.0.251.246;
 
        location / {
            # 以实际部署的后台服务地址为准
            proxy_pass https://192.168.32.34:8088;
        }
 
    }
}
```

## 在 conf 文件夹内创建 mime.types 文件。


```

types {
    text/html                                        html htm shtml;
    text/css                                         css;
    text/xml                                         xml;
    image/gif                                        gif;
    image/jpeg                                       jpeg jpg;
    application/javascript                           js;
    application/atom+xml                             atom;
    application/rss+xml                              rss;

    text/mathml                                      mml;
    text/plain                                       txt;
    text/vnd.sun.j2me.app-descriptor                 jad;
    text/vnd.wap.wml                                 wml;
    text/x-component                                 htc;

    image/png                                        png;
    image/svg+xml                                    svg svgz;
    image/tiff                                       tif tiff;
    image/vnd.wap.wbmp                               wbmp;
    image/webp                                       webp;
    image/x-icon                                     ico;
    image/x-jng                                      jng;
    image/x-ms-bmp                                   bmp;

    application/font-woff                            woff;
    application/java-archive                         jar war ear;
    application/json                                 json;
    application/mac-binhex40                         hqx;
    application/msword                               doc;
    application/pdf                                  pdf;
    application/postscript                           ps eps ai;
    application/rtf                                  rtf;
    application/vnd.apple.mpegurl                    m3u8;
    application/vnd.google-earth.kml+xml             kml;
    application/vnd.google-earth.kmz                 kmz;
    application/vnd.ms-excel                         xls;
    application/vnd.ms-fontobject                    eot;
    application/vnd.ms-powerpoint                    ppt;
    application/vnd.oasis.opendocument.graphics      odg;
    application/vnd.oasis.opendocument.presentation  odp;
    application/vnd.oasis.opendocument.spreadsheet   ods;
    application/vnd.oasis.opendocument.text          odt;
    application/vnd.openxmlformats-officedocument.presentationml.presentation
                                                     pptx;
    application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
                                                     xlsx;
    application/vnd.openxmlformats-officedocument.wordprocessingml.document
                                                     docx;
    application/vnd.wap.wmlc                         wmlc;
    application/x-7z-compressed                      7z;
    application/x-cocoa                              cco;
    application/x-java-archive-diff                  jardiff;
    application/x-java-jnlp-file                     jnlp;
    application/x-makeself                           run;
    application/x-perl                               pl pm;
    application/x-pilot                              prc pdb;
    application/x-rar-compressed                     rar;
    application/x-redhat-package-manager             rpm;
    application/x-sea                                sea;
    application/x-shockwave-flash                    swf;
    application/x-stuffit                            sit;
    application/x-tcl                                tcl tk;
    application/x-x509-ca-cert                       der pem crt;
    application/x-xpinstall                          xpi;
    application/xhtml+xml                            xhtml;
    application/xspf+xml                             xspf;
    application/zip                                  zip;

    application/octet-stream                         bin exe dll;
    application/octet-stream                         deb;
    application/octet-stream                         dmg;
    application/octet-stream                         iso img;
    application/octet-stream                         msi msp msm;

    audio/midi                                       mid midi kar;
    audio/mpeg                                       mp3;
    audio/ogg                                        ogg;
    audio/x-m4a                                      m4a;
    audio/x-realaudio                                ra;

    video/3gpp                                       3gpp 3gp;
    video/mp2t                                       ts;
    video/mp4                                        mp4;
    video/mpeg                                       mpeg mpg;
    video/quicktime                                  mov;
    video/webm                                       webm;
    video/x-flv                                      flv;
    video/x-m4v                                      m4v;
    video/x-mng                                      mng;
    video/x-ms-asf                                   asx asf;
    video/x-ms-wmv                                   wmv;
    video/x-msvideo                                  avi;
}

```


## 启动nginx


```
# nginx启动命令
nginx -c /usr/share/nginx/conf/nginx.conf
```

**常用nginx命令**


```
# nginx 重启
nginx -s reload
# nginx 停止
ps -ef | grep nginx      //查看nginx当前启动进程
kill -9 （进程ID）        //当前nginx的进程ID
```


