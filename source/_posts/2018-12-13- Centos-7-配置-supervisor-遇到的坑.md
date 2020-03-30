---
title:  Centos-7-配置-supervisor-遇到的坑
date: 2018-12-13 21:19:03
tags: 
- supervisor
categories: Server
---

@[toc]

---
环境 centos7.5  
## 1、安装supervisor
```shell
yum install supervisor
执行完之后 在/etc 目录中会生成 supervisord.conf
```
## 2、配置
![supervisor](/img/image/supervisor.png)

> 如上图所示   这里需要在 /etc/supervisord.d/  目录中创建你需要的配置文件

> 我创建如下

```shell
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /data/wwwroot/default/coin/artisan queue:work  --sleep=3 --tries=3
autostart=true
autorestart=true
user=root
numprocs=8
redirect_stderr=true
stdout_logfile=/data/wwwroot/default/coin/storage/logs/worker.log
```

>配置文件创建完成后
执行

```shell
 [root@VM_0_7_centos etc]# supervisorctl reload
error: <class 'socket.error'>, [Errno 111] Connection refused: file: /usr/lib64/python2.7/socket.py line: 224
```
>解决办法： 
这个可能有多种原因，可能是已经启动过了也可能是没权限，解决步骤如下：

### 1、先要确认是否已经启动过了：ps -ef | grep supervisord
### 2、如果有的话先kill掉 
### 3、运行下面命令： 

```shell
touch /var/run/supervisor.sock 
chmod 777 /var/run/supervisor.sock 
```

### 4、再执行如下命令

```shell
[root@VM_0_7_centos etc]# supervisord -c supervisord.conf 
Unlinking stale socket /var/run/supervisor/supervisor.sock
[root@VM_0_7_centos etc]# unlink /var/run/supervisor/supervisor.sock
[root@VM_0_7_centos etc]# ps -ef | grep supervisor
root     13704     1  1 14:08 ?        00:00:03 /usr/bin/python /usr/bin/supervisord -c supervisord.conf
root     25913 32596  0 14:14 pts/0    00:00:00 grep --color supervisor
root     32227     1  0 11:54 ?        00:00:01 /usr/bin/python /usr/bin/supervisord

```
## 3、启动

```shell
supervisord -c supervisord.conf
```

>我这启动时报错如下

```shell
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.
For help, use /usr/bin/supervisord -h
```

>解决办法:

```shell
[root@VM_0_7_centos etc]# supervisord -c supervisord.conf 
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.
For help, use /usr/bin/supervisord -h
[root@VM_0_7_centos etc]# ps -ef | grep supervisor
root     10276 32596  0 13:46 pts/0    00:00:00 grep --color supervisor
root     32227     1  0 11:54 ?        00:00:00 /usr/bin/python /usr/bin/supervisord
[root@VM_0_7_centos etc]# find / -name supervisor.sock
/run/supervisor/supervisor.sock
[root@VM_0_7_centos etc]# unlink /name/supervisor.sock
unlink: cannot unlink ‘/name/supervisor.sock’: No such file or directory
[root@VM_0_7_centos etc]# unlink /run/supervisor/supervisor.sock
[root@VM_0_7_centos etc]# supervisord -c supervisord.conf 
[root@VM_0_7_centos etc]# ps -ef | grep supervisor
root     10609     1  1 13:49 ?        00:00:00 /usr/bin/python /usr/bin/supervisord -c supervisord.conf
root     12309 32596  0 13:49 pts/0    00:00:00 grep --color supervisor
root     32227     1  0 11:54 ?        00:00:00 /usr/bin/python /usr/bin/supervisord

```
## 4、总结 

>常用命令

```shell
#启动守护进程：
supervisord -c /etc/supervisor/supervisord.conf
#重载配置：
supervisorctl reload

sudo supervisorctl reread
sudo supervisorctl update
```

```
报错信息： 
Error: .ini file does not include supervisorctl section
解决办法： 
一个权限问题，步骤： 
1. 先确保supervisord.conf中有[supervisord]，[supervisorctl]有这两个基本模块，还有[program:XXX]自己程序的配置（可以在supervisord.conf也可以在/etc/supervisor/*.ini中） 
2. 最关键的两个命令：chmod +x /usr/bin/supervisord 
chmod +x /usr/bin/supervisorctl 
chmod +x /etc/supervisord.conf主要是把把相关文件都授权了 
3. 把supervisord杀掉后再次启动supervisord
```
```
Exited too quickly (process log may have details)

解决办法： 
1. 先确认[program:XXX]中自己的程序的command=<启动命令>和 directory=<运行命令的路径>没有问题，python是不是用的自己要的环境的python（比如虚拟环境的），log文件的文件夹是不是已经创建（没创建的话supervisor没权限生成log文件），以及改log文件是不是授权给所有用户了（可参考前面的解决办法chmod +x aaaaa.log） 
2. 确保用上面的配置中的command在指定路径可以直接运行不会报错，这时候一般就不会有什么问题了。这时候tail你自己的log文件一般就能看到log信息，启动失败报错的信息也会在你的log文件中，照着解决后supervisorctl reload就好了。 
3. 如果上面的命令确保可以跑，但还是没法正常运行，也看不到自己程序的报错（不然你就能根据报错解决问题了），那么恭喜，你遇到了跟我一样的情况。我的解决办法很诡异，尝试把[program:XXX]中的名字换成了一个跟启动命令不一样的另一个名字（不要太短），reload之后居然就可以跑了
```

```
还有一个比较坑的  如果你是直接把laravel官方文档中的supervisor配置拿过来的话 一定记得改驱动如下图所示
```
![supervisor](/img/image/supervisor2.png)


> 我就是直接复制过来的 没改驱动  结果日志一直报错 如下

```php
[2018-12-13 14:11:06] local.ERROR: Class 'Aws\Sqs\SqsClient' not found {"exception":"[object] (Symfony\\Component\\Debug\\Exception\\FatalThrowableError(code: 0): Class 'Aws\\Sqs\\SqsClient' not found at /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Queue/Connectors/SqsConnector.php:26)
[stacktrace]
#0 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Queue/QueueManager.php(157): Illuminate\\Queue\\Connectors\\SqsConnector->connect(Array)
#1 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Queue/QueueManager.php(138): Illuminate\\Queue\\QueueManager->resolve('sqs')
#2 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Queue/Worker.php(105): Illuminate\\Queue\\QueueManager->connection('sqs')
#3 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Queue/Console/WorkCommand.php(101): Illuminate\\Queue\\Worker->daemon('sqs', 'your-queue-name', Object(Illuminate\\Queue\\WorkerOptions))
#4 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Queue/Console/WorkCommand.php(85): Illuminate\\Queue\\Console\\WorkCommand->runWorker('sqs', 'your-queue-name')
#5 [internal function]: Illuminate\\Queue\\Console\\WorkCommand->handle()
#6 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Container/BoundMethod.php(29): call_user_func_array(Array, Array)
#7 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Container/BoundMethod.php(87): Illuminate\\Container\\BoundMethod::Illuminate\\Container\\{closure}()
#8 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Container/BoundMethod.php(31): Illuminate\\Container\\BoundMethod::callBoundMethod(Object(Illuminate\\Foundation\\Application), Array, Object(Closure))
#9 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Container/Container.php(549): Illuminate\\Container\\BoundMethod::call(Object(Illuminate\\Foundation\\Application), Array, Array, NULL)
#10 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Console/Command.php(183): Illuminate\\Container\\Container->call(Array)
#11 /data/wwwroot/default/coin/vendor/symfony/console/Command/Command.php(255): Illuminate\\Console\\Command->execute(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Illuminate\\Console\\OutputStyle))
#12 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Console/Command.php(170): Symfony\\Component\\Console\\Command\\Command->run(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Illuminate\\Console\\OutputStyle))
#13 /data/wwwroot/default/coin/vendor/symfony/console/Application.php(953): Illuminate\\Console\\Command->run(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Symfony\\Component\\Console\\Output\\ConsoleOutput))
#14 /data/wwwroot/default/coin/vendor/symfony/console/Application.php(248): Symfony\\Component\\Console\\Application->doRunCommand(Object(Illuminate\\Queue\\Console\\WorkCommand), Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Symfony\\Component\\Console\\Output\\ConsoleOutput))
#15 /data/wwwroot/default/coin/vendor/symfony/console/Application.php(148): Symfony\\Component\\Console\\Application->doRun(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Symfony\\Component\\Console\\Output\\ConsoleOutput))
#16 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Console/Application.php(88): Symfony\\Component\\Console\\Application->run(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Symfony\\Component\\Console\\Output\\ConsoleOutput))
#17 /data/wwwroot/default/coin/vendor/laravel/framework/src/Illuminate/Foundation/Console/Kernel.php(121): Illuminate\\Console\\Application->run(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Symfony\\Component\\Console\\Output\\ConsoleOutput))
#18 /data/wwwroot/default/coin/artisan(37): Illuminate\\Foundation\\Console\\Kernel->handle(Object(Symfony\\Component\\Console\\Input\\ArgvInput), Object(Symfony\\Component\\Console\\Output\\ConsoleOutput))
#19 {main}

```

## 参考文章
[supervisor常见报错](https://blog.csdn.net/kkevinyang/article/details/80539940)

---
