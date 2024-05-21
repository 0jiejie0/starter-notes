# post请求失败failed The system cannot find the path specified

## 背景

拆分项目，在nginx服务器新增一个项目，vue-springboot架构。

## 问题描述

```NGINX
server {
    listen       91;
    server_name  localhost;
    location / {
        root hzbzz;
        index index.html index.htm;
    }
}
```

如上配置好nginx静态路径，localhost:91页面可以打开，但是却登录不上，打开f12发现login请求出问题了：

```log
[error] 4836#2240: *128768 CreateFile() "./hzbzz/ls/uuu/login" failed (3: The system cannot find the path specified), client: 127.0.0.1, server: localhost, request: "POST /ls/uuu/login HTTP/1.1", host: "localhost:91", referrer: "http://localhost:91/login"
```

## 原因分析

感觉像是post转发请求，路径匹配出了问题，从另外可用的主机配置中找到这么一段：

```NGINX
    location /ls/uuu {
        proxy_pass http://api_server;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```

## 解决方案

把location添加到主机配置中：

```NGINX
server {
    listen       91;
    server_name  localhost;
    location / {
        root hzbzz;
        index index.html index.htm;
    }
    location /ls/uuu {
        proxy_pass http://api_server;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

成功登陆。
