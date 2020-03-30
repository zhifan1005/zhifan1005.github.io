---
title:  基于 MySQL 协议，Swoole 开发的MySQL数据库连接池使用教程。
date: 2019-12-17 13:50:38
tags: 
- SMProxy 
- mysql 
categories: Mysql
---

@[toc]

>***基于 MySQL 协议，Swoole 开发的MySQL数据库连接池使用教程***

---
## 安装

```
git clone https://github.com/louislivi/SMProxy.git
composer install --no-dev # 如果你想贡献你的代码，请不要使用 --no-dev 参数。
```

---
## 运行

```
cd SMProxy
bin/SMProxy start 
```
>启动成功后的界面如下所示

```
root@JD:/var/www/SMProxy# bin/SMProxy start

  /$$$$$$  /$$      /$$ /$$$$$$$                                        
 /$$__  $$| $$$    /$$$| $$__  $$                                       
| $$  \__/| $$$$  /$$$$| $$  \ $$ /$$$$$$   /$$$$$$  /$$   /$$ /$$   /$$
|  $$$$$$ | $$ $$/$$ $$| $$$$$$$//$$__  $$ /$$__  $$|  $$ /$$/| $$  | $$
 \____  $$| $$  $$$| $$| $$____/| $$  \__/| $$  \ $$ \  $$$$/ | $$  | $$
 /$$  \ $$| $$\  $ | $$| $$     | $$      | $$  | $$  >$$  $$ | $$  | $$
|  $$$$$$/| $$ \/  | $$| $$     | $$      |  $$$$$$/ /$$/\  $$|  $$$$$$$
 \______/ |__/     |__/|__/     |__/       \______/ |__/  \__/ \____  $$
                                                               /$$  | $$
                                                              |  $$$$$$/
                                                               \______/
                                                               

SMProxy version: v1.3.0@e6a640a

Server starting ...
```
>查看启动后的状态

```
root@JD:/var/www/SMProxy# bin/SMProxy status
SMProxy[v1.3.0] - Linux JD 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64
Host: 0.0.0.0, Port: 3366, PHPVerison: 7.3.12-1+ubuntu16.04.1+deb.sury.org+1
SwooleVersion: 4.4.12, WorkerNum: 2
Process :  26 total,  24 sleep,  2 query
+------+--------+-----------------+-----------+---------+------+-----------+-----------------------------+-------------------------+-----------------------+---------------+-------------+
| ID   | USER   | HOST            | DB        | COMMAND | TIME | STATE     | INFO                        | SERVER_VERSION          | PLUGIN_NAME           | SERVER_STATUS | SERVER_KEY  |
+------+--------+-----------------+-----------+---------+------+-----------+-----------------------------+-------------------------+-----------------------+---------------+-------------+
| 1789 | zhifan | localhost:58178 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | writeSΜbbs |
| 1790 | zhifan | localhost:58182 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | writeSΜbbs |
| 1791 | zhifan | localhost:58186 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | writeSΜbbs |
| 1793 | zhifan | localhost:58190 | bbs       | Query   | 0    | executing | /*SMProxy processlist sql*/ | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | writeSΜbbs |
| 1794 | zhifan | localhost:58196 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1795 | zhifan | localhost:58200 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1796 | zhifan | localhost:58204 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1797 | zhifan | localhost:58208 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1798 | zhifan | localhost:58212 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1799 | zhifan | localhost:58216 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1800 | zhifan | localhost:58220 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1801 | zhifan | localhost:58224 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1802 | zhifan | localhost:58228 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1803 | zhifan | localhost:58232 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1804 | zhifan | localhost:58236 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1805 | zhifan | localhost:58240 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1806 | zhifan | localhost:58244 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1807 | zhifan | localhost:58248 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1808 | zhifan | localhost:58252 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1809 | zhifan | localhost:58270 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1810 | zhifan | localhost:58268 | bbs       | Query   | 0    | executing | /*SMProxy processlist sql*/ | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1811 | zhifan | localhost:58264 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1812 | zhifan | localhost:58260 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1813 | zhifan | localhost:58256 | bbs       | Sleep   | 35   |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | readSΜbbs  |
| 1814 | zhifan | localhost:58278 | bookstack | Sleep   | 6    |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | read        |
| 1815 | zhifan | localhost:58280 | bookstack | Sleep   | 6    |           |                             | 5.7.28-0ubuntu0.16.04.2 | mysql_native_password | 2             | read        |
+------+--------+-----------------+-----------+---------+------+-----------+-----------------------------+-------------------------+-----------------------+---------------+-------------+
```
---
## 我遇到的问题

```
2019-12-17 13:37:21 [warning] Config serverInfo->write->account is not exists! (/var/www/SMProxy/src/Base.php:99)
2019-12-17 13:37:21 [warning] Config serverInfo->write->account is not exists! (/var/www/SMProxy/src/Base.php:99)
2019-12-17 13:38:35 [info] Worker started!
```
[常见问题](https://smproxy.louislivi.com/#/README?id=%e5%b8%b8%e8%a7%81%e9%97%ae%e9%a2%98)

>参考上面的说明 我把database->account下面的root 改为了zhifan 然后启动成功

>这里附上我自己的配置文件 对于account对应的用户名非root远程访问的一定要记得修改root那个key为自己账户的

## 我的database.json文件如下


```
{
  "database": {
    "account": {
      "db_user": { # 我就是错在这里了  之前是root  我忘记修改 导致报错 改完之后即可正常运行
        "zhifan": "zhifan", 
        "password": "password"
      }
    },
    "serverInfo": {
      "server1": {
        "write": {
          "host": ["127.0.0.1"],
          "port": 3306,
          "timeout": 2,
          "account": "zhifan"
        },
        "read": {
          "host": ["127.0.0.1"],
          "port": 3306,
          "timeout": 2,
          "account": "zhifan",
          "startConns": "swoole_cpu_num()*10",
          "maxSpareConns": "swoole_cpu_num()*10",
          "maxSpareExp": 3600,
          "maxConns": "swoole_cpu_num()*20"
        }
      }
    },
    "databases": {
      "bbs": {
        "serverInfo": "server1",
        "startConns": "swoole_cpu_num()*2",
        "maxSpareConns": "swoole_cpu_num()*2",
        "maxSpareExp": 3600,
        "maxConns": "swoole_cpu_num()*2",
        "charset": "utf8mb4"
      }
    }
  }
}
```

>该命令会发布配置文件 laravels.php 到 config 目录下，以及脚本文件到 bin 目录下：

## 我的server.json文件如下

```
{
  "server": {
    "user": "root",
    "password": "123456",
    "charset": "utf8mb4",
    "host": "0.0.0.0",
    "port": "3366",
    "mode": "SWOOLE_PROCESS",
    "sock_type": "SWOOLE_SOCK_TCP",
    "logs": {
      "open":true,
      "config": {
        "system": {
          "log_path": "ROOT/logs",
          "log_file": "system.log",
          "format": "Y/m/d"
        },
        "mysql": {
          "log_path": "ROOT/logs",
          "log_file": "mysql.log",
          "format": "Y/m/d"
        }
      }
    },
    "swoole": {
      "worker_num": "swoole_cpu_num()",
      "max_coro_num": 6000,
      "open_tcp_nodelay": true,
      "daemonize": true,
      "heartbeat_check_interval": 60,
      "heartbeat_idle_time": 600,
      "reload_async": true,
      "log_file": "ROOT/logs/swoole.log",
      "pid_file": "ROOT/logs/pid/server.pid"
    },
    "swoole_client_setting": {
      "package_max_length": 16777215
    },
    "swoole_client_sock_setting": {
      "sock_type": "SWOOLE_SOCK_TCP"
    }
  }
}
```
## 官网说明
[参考官网链接](https://smproxy.louislivi.com/#/README)
