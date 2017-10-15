---
layout: page
title: "Nginx"
date: 2014-01-08 10:30
comments: true
sharing: true
footer: true
---

## 使用入门

``` sh
# 安装nginx
apt-get install nginx

# 启动nginx
service nginx start

# 停止nginx
service nginx stop

# 重启nginx
service nginx restart

# 重新加载配置
service nginx reload

# 强制重新加载配置
service nginx force-reload

# 查看nginx运行状态
service nginx status

# 测试配置(合法性)
service nginx configtest
```

## 配置

### 反向代理

```
server {
    # 监听端口
    listen 80;
    # 代理对外域名
    server_name proxy-site.com;

    location / {
        # 转向服务器
        proxy_pass http://dest-site.com;
        proxy_redirect default;
    }
}

# 服务器集群及权重(可选)
upstream dest-site.com {
    server 10.0.0.1:80 weight=1;
}
```

### 静态资源缓存

```
server {
    listen 80;
    server_name some-site.com;

    root /some/path/to/site;

    # 缓存图片
    location ~ \.(jpg|png|jpeg|bmp|gif|swf)$ {
        root /some/path/to/site/images;
        if (-f $request_filename) {
            expires 7d;
            break;
        }
    }

    # 缓存样式
    location ~ \.(css)$ {
        root /some/path/to/site/css;
        if (-f $request_filename) {
            expires 3d;
            break;
        }
    }

    # 缓存脚本
    location ~ \.(js)$ {
        root /some/path/to/site/js;
        if (-f $request_filename) {
            expires 1d;
            break;
        }
    }
}
```

### 变量

|变量名|作用|
|:----|:----|
|$args|请求中的参数|
|$content_length|HTTP请求头Content-Length|
|$content_type|HTTP请求头Content-Type|
|$document_root|root路径|
|\$document_uri|同$request_uri|
|$limit_rate|连接速率限制|
|$remote_addr|客户端地址|
|$remote_port|客户端端口|
|$remote_user|认证用户名|
|$request_filename|请求文件路径|
|$request_body_file||
|$request_method|方法|
|$request_uri|当前URI，只包含主机名后部分|
|\$query_string|同$args|
|$schema|协议|
|$server_addr|服务器地址|
|$server_name|服务器名|
|$server_port|服务器端口|
|\$uri|同$request_uri|
