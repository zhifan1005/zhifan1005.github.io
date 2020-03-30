---
title:  在 Laravel 中集成 Swoole 实现 WebSocket 服务器
date: 2019-12-11 20:45:19
tags: 
- swoole 
- laravel 
- websocket
categories: Swoole
---

@[toc]

>***在 Laravel 中集成 Swoole 实现 WebSocket 服务器***

---
## 1.创建 App/Services/WebSocketService.php文件


```php
<?php

namespace App\Services;

use Hhxsv5\LaravelS\Swoole\WebSocketHandlerInterface;
use Illuminate\Support\Facades\Log;
use Swoole\Http\Request;
use Swoole\WebSocket\Frame;
use Swoole\WebSocket\Server;

class WebSocketService implements WebSocketHandlerInterface
{
    public function __construct()
    {

    }

    // 连接建立时触发
    public function onOpen(Server $server, Request $request)
    {
        // 在触发 WebSocket 连接建立事件之前，Laravel 应用初始化的生命周期已经结束，你可以在这里获取 Laravel 请求和会话数据
        // 调用 push 方法向客户端推送数据，fd 是客户端连接标识字段
        Log::info('WebSocket 连接建立');
        $server->push($request->fd, 'Welcome to WebSocket Server built on LaravelS');
    }

    // 收到消息时触发
    public function onMessage(Server $server, Frame $frame)
    {
        // 调用 push 方法向客户端推送数据
        $server->push($frame->fd, "You say {".$frame->data ."}at " . date('Y-m-d H:i:s'));
    }

    // 关闭连接时触发
    public function onClose(Server $server, $fd, $reactorId)
    {
        Log::info('WebSocket 连接关闭');
    }
}
```
---
## 2. 更改配置config/laravels.php
```php
// ...
'websocket'      => [
    'enable'  => true, // 看清楚，这里是true
    'handler' => \App\Services\WebSocketService::class,
],
'swoole'         => [
    //...
    // dispatch_mode只能设置为2、4、5，https://wiki.swoole.com/wiki/page/277.html
    'dispatch_mode' => 2,
    //...
],
// ...
```
---
## 3. 配置 Nginx 支持 WebSocket
```nginx
 map $http_upgrade $connection_upgrade {
     default upgrade;
     ''      close;
 }
 upstream swoole {
     # 通过 IP:Port 连接
     server 127.0.0.1:5200 weight=5 max_fails=3 fail_timeout=30s;
     # 通过 UnixSocket Stream 连接，小诀窍：将socket文件放在/dev/shm目录下，可获得更好的性能
     #server unix:/xxxpath/laravel-s-test/storage/laravels.sock weight=5 max_fails=3 fail_timeout=30s;
     #server 192.168.1.1:5200 weight=3 max_fails=3 fail_timeout=30s;
     #server 192.168.1.2:5200 backup;
     keepalive 16;
 }
 server {
     listen 80;
     # 别忘了绑Host哟
     server_name laravels.com;
     root /xxxpath/laravel-s-test/public;
     access_log /yyypath/log/nginx/$server_name.access.log  main;
     autoindex off;
     index index.html index.htm;
     # Nginx处理静态资源(建议开启gzip)，LaravelS处理动态资源。
     location / {
         try_files $uri @laravels;
     }
     # 当请求PHP文件时直接响应404，防止暴露public/*.php
     #location ~* \.php$ {
     #    return 404;
     #}
     # Http和WebSocket共存，Nginx通过location区分
     # !!! WebSocket连接时路径为/ws
     # Javascript: var ws = new WebSocket("ws://laravels.com/ws");
     location =/ws {
         # proxy_connect_timeout 60s;
         # proxy_send_timeout 60s;
         # proxy_read_timeout：如果60秒内被代理的服务器没有响应数据给Nginx，那么Nginx会关闭当前连接；同时，Swoole的心跳设置也会影响连接的关闭
         # proxy_read_timeout 60s;
         proxy_http_version 1.1;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Real-PORT $remote_port;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header Host $http_host;
         proxy_set_header Scheme $scheme;
         proxy_set_header Server-Protocol $server_protocol;
         proxy_set_header Server-Name $server_name;
         proxy_set_header Server-Addr $server_addr;
         proxy_set_header Server-Port $server_port;
         proxy_set_header Upgrade $http_upgrade;
         proxy_set_header Connection $connection_upgrade;
         proxy_pass http://swoole;
     }
     location @laravels {
         # proxy_connect_timeout 60s;
         # proxy_send_timeout 60s;
         # proxy_read_timeout 60s;
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
         proxy_pass http://swoole;
     }
 }
```

## 4.重载配置并重启Laravels服务

```
 php bin/laravels reload
 php bin/laravels restart
```
>这里切记一定要重启 否则websocket无法连接上

## 5.配置wss服务

>这里只需要把你自己配置的域名升级为https即可 亲测有效

![第三方测试如下](/img/image/websocket_test.png)

## 6.配置测试页面

### 6-1 在web.php新增路由

```php
Route::get('/ws', function () {
    return view('ws');
});
```

### 6-2 在resources/views新增ws.blade.php

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Chat Client</title>
</head>
<body>
<script>
    window.onload = function () {
        var nick = prompt("Enter your nickname");
        var input = document.getElementById("input");
        input.focus();

        // 初始化客户端套接字并建立连接
        var socket = new WebSocket("wss://你的域名/ws");

        // 连接建立时触发
        socket.onopen = function (event) {
            console.log("Connection open ...");
        }

        // 接收到服务端推送时执行
        socket.onmessage = function (event) {
            var msg = event.data;
            var node = document.createTextNode(msg);
            var div = document.createElement("div");
            div.appendChild(node);
            document.body.insertBefore(div, input);
            input.scrollIntoView();
        };

        // 连接关闭时触发
        socket.onclose = function (event) {
            console.log("Connection closed ...");
        }

        input.onchange = function () {
            var msg = nick + ": " + input.value;
            // 将输入框变更信息通过 send 方法发送到服务器
            socket.send(msg);
            input.value = "";
        };
    }
</script>
<input id="input" style="width: 100%;">
</body>
</html>
```

### 6-3 在页面测试如下

>测试页面效果

![页面测试如下](/img/image/websocket_test2.png)

>握手过程

![握手过程](/img/image/websocket.jpg)


