---
title:  基于swoole实现HTTP高性能服务器
date: 2019-12-11 13:59:12
tags: 
- swoole 
- http 
- supervisor
categories: Swoole
---

@[toc]

>***基于swoole实现HTTP高性能服务器***

---
## 1.拉取一个新的laravel框架 

```
laravel new laravelSwoole
```
---
## 2. 安装配置 LaravelS
```
composer require hhxsv5/laravel-s
```
---
## 3. 发布配置
```
 php artisan laravels publish
```
>该命令会发布配置文件 laravels.php 到 config 目录下，以及脚本文件到 bin 目录下：

## 4.启动 LaravelS

```
 php bin/laravels start
```
>启动Swoole服务，并且监听本地的5200端口，如果有请求发送到这个端口，它就可以进行处理

![界面如下](/img/image/laravels_start.png)

>此外 php bin/laravels 还支持其它命令对 LaravelS 进行管理：

![命令如下](/img/image/laravels_help.png)

## 5.通过 Supervisor 管理 LaravelS
### 5-1 安装Supervisor
>这里我是用的ubuntu系统。centos系统的话请看我这篇文章[Centos-7-配置-supervisor](https://renzhifan.github.io/tool/Centos-7-%E9%85%8D%E7%BD%AE-supervisor-%E9%81%87%E5%88%B0%E7%9A%84%E5%9D%91/)

```
 sudo apt-get install supervisor
 cd /etc/supervisor/conf.d
 touch laravel-s-swoole.conf 
```

>laravel-s-swoole.conf 内容如下

```
[program:laravel-s-swoole]
command=php /var/www/laravelSwoole/bin/laravels restart -i
numprocs=1
autostart=true
autorestart=true
startretries=3
user=www-data
redirect_stderr=true
stdout_logfile=/var/www/laravelSwoole/storage/logs/supervisord-stdout.log
```
>Supervisor 重载配置:
 
```
supervisorctl reload
```

### 5-2  配置 Nginx

>我们知道在使用 Nginx 作为 Web 服务器的时候，前端资源文件，比如 CSS、JS、图片等静态资源都是通过 Nginx 进行处理的，比较高效，而 PHP 脚本请求这种动态资源都是转发到后端 PHP-FPM 进程进行处理，如果要基于 Swoole 实现高性能 HTTP 服务器，则这个 HTTP 服务器替代的也是 PHP-FPM 的职能，也就是说，我们将原本转发到 PHP-FPM 进程的请求转发给 Swoole 进行处理。在本例中，就是转发给 LaravelS 服务

```
upstream laravels {
    # Connect IP:Port
    server 127.0.0.1:5200 weight=5 max_fails=3 fail_timeout=30s;
    keepalive 16;
}
server {
    listen 80;
    
    server_name 你的域名;
    root /var/www/laravelSwoole/public;
    index index.php index.html index.htm;
    
    # Nginx 处理静态资源，LaravelS 处理动态资源
    location / {
        try_files $uri @laravels;
    }
    
    location @laravels {
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-PORT $remote_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header Server-Protocol $server_protocol;
        proxy_set_header Server-Name $server_name;
        proxy_set_header Server-Addr $server_addr;
        proxy_set_header Server-Port $server_port;
        proxy_pass http://laravels;
    }
}
```

### 5-3.配置 Laravel 应用

>在你的env配置文件增加两行

```
LARAVELS_LISTEN_IP=127.0.0.1
LARAVELS_DAEMONIZE=true
```
### 5-4 测试一下

>访问你自己配置的域名能正常出现laravel的首页表示配置成功

