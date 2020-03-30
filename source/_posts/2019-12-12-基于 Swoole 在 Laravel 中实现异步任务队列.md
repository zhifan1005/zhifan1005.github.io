---
title:  基于 Swoole 在 Laravel 中实现异步任务队列
date: 2019-12-12 21:11:19
tags: 
- swoole 
- laravel 
categories: Swoole
---

@[toc]

>***基于 Swoole 在 Laravel 中实现异步任务队列***

---
## 1.实现原理 

>PHP 本身的设计是同步阻塞的，不支持多线程和异步 IO，所以当我们执行一些耗时的操作，
比如发送广播，或者邮件，如果直接在当前进程中操作，会导致服务器响应变慢，
因此要借助一些第三方服务来处理以实现异步功能，
比如队列，而 Swoole 作为 PHP 异步网络通信引擎，
自然也对异步任务处理提供了支持，
其底层的实现原理和常见的异步队列类似：
将耗时任务投递到 TaskWorker 进程池后返回
（相应任务会通过 TaskWorker 异步执行，执行成功后可以调用事先注册的回调函数进行后续处理），
继续后续业务逻辑的执行，而不影响当前请求的处理速度。

---
## 2. 编写任务类 App\Jobs\TestTask.php
```php
<?php
namespace App\Jobs;

use Hhxsv5\LaravelS\Swoole\Task\Task;
use Illuminate\Support\Facades\Log;

class TestTask extends Task
{
    // 待处理任务数据
    private $data;

    // 任务处理结果
    private $result;

    public function __construct($data)
    {
        $this->data = $data;
    }

    // 任务投递调用 task 回调时触发，等同于 Swoole 中的 onTask 逻辑
    public function handle()
    {
        Log::info(__CLASS__ . ': 开始处理任务', [$this->data]);
        //  todo 耗时任务具体处理逻辑在这里编写
        sleep(3); // 模拟任务需要3秒才能执行完毕
        $this->result = 'The result of ' . $this->data . ' is balabalabala';
    }

    // 任务完成调用 finish 回调时触发，等同于 Swoole 中的 onFinish 逻辑
    // 可选的，完成事件，任务处理完后的逻辑，运行在Worker进程中，可以投递任务
    public function finish()
    {
        \Log::info(__CLASS__ . ':finish start', [$this->result]);
//        Task::deliver(new TestTask2('task2')); // 投递其他任务
    }
}
```
---
## 3. 编写测试代码

>然后在 routes/web.php 编写投递异步任务的测试代码如下：

```php
use Hhxsv5\LaravelS\Swoole\Task\Task;
use App\Jobs\TestTask;
Route::get('/task/test', function () {
    $task = new TestTask('task data');
// $task->delay(3);// 延迟3秒投放任务
    $ret = Task::deliver($task);
    var_dump($ret);//判断是否投递成功
});
```
>该命令会发布配置文件 laravels.php 到 config 目录下，以及脚本文件到 bin 目录下：

## 4.修改配置文件

>在配置文件 config/laravels.php 中取消 task_worker_num 配置项前面的注释：

```php
 'swoole' => [
     ...
     'task_worker_num'    => function_exists('swoole_cpu_num') ? swoole_cpu_num() * 2 : 8,
     ...
 ]
```

## 5.测试异步任务执行

>***接下来，我们重启启动 Swoole 服务器（基于 Swoole HTTP 服务器访问路由才能成功投递异步任务）：***

```
php bin/laravels restart
```

>然后在浏览器中通过 http://你的域名/task/test 访问测试路由，页面立即显示投递成功：

```
bool(true)
```
>然后我们去 storage/logs 目录下查看最新的日志信息，可以看到任务执行其实耗费了 3 秒：

![如图所示](/img/image/swoole_asynchronous.png)
