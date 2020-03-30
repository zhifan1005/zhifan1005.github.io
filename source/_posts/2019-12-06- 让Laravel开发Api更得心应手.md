---
title:  让Laravel开发Api更得心应手
date: 2019-12-11 13:55:16
tags: 
- api 
- jwt 
- horizon
categories: Laravel
---

@[toc]

---

## 前言
先把源码扔出来 可进行对比参考 [手摸手教你让Laravel开发Api更得心应手](https://github.com/renzhifan/laravelApi)

---
## 1.安装Laravel
```

laravel new laravelApi (这里我使用的laravel安装器 默认安装最新版本)
如果要安装指定版本的话建议使用composer
composer create-project laravel/laravel Laravel --prefer-dist "6.0.*"

```

## 2.建立数据库
```php
php artisan make:migration create_users_table(里面内容可参考我对应的文件)

```
## 3.移动User模型

### 3-1. 在app目录下新建Models文件夹，然后将User.php文件移动进来。

### 3-2.  修改User.php的内容
```php

<?php

namespace App\Models; //这里从App改成了App\Models

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;
    protected $table = 'users';

     //去掉我创建的数据表没有的字段
    protected $fillable = [
        'name', 'password'
    ];

     //去掉我创建的数据表没有的字段
    protected $hidden = [
        'password'
    ];
    //将密码进行加密
    public function setPasswordAttribute($value)
    {
        $this->attributes['password'] = bcrypt($value);
    }
}

```

### 3-3.  因为有关于User的命名空间发生了改变，所以我们全局搜索App\User,将其替换为App\Models\User我一共搜索到3个文件
```
app/Http/Controllers/Auth 目录下的 RegisterController.php
config 目录下的 services.php
config 目录下的 auth.php
database/factories 目录下的 UserFactory.php

```
## 4.创建对应的控制器路由以及对应的验证器（可参考源码）

## 5.生成一批测试数据 可参考我seed里面对应的文件

```
php artisan db:seed;
```

## 6.跨域问题
### 6-1.安装medz/cors
```
composer require medz/cors

```

### 6-2.发布配置文件
```

php artisan vendor:publish --provider="Medz\Cors\Laravel\Providers\LaravelServiceProvider" --force

```

### 6-3. 修改配置文件
```
return [
    ......
    'expose-headers'     => ['Authorization'],
    ......
];

```

>这样跨域请求时，才能返回header头为Authorization的内容，否则在刷新用户token时不会返回刷新后的token


### 6-4.增加中间件别名

打开App\Http下的Kernel文件
      
```php
protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'password.confirm' => \Illuminate\Auth\Middleware\RequirePassword::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
        ...... //前面的中间件
        'cors'=> \Medz\Cors\Laravel\Middleware\ShouldGroup::class,
];
```

### 6-5. 修改路由(增加刚才的中间件)


## 7.统一Response响应处理[参考这里](https://learnku.com/articles/6035/laravel55-developing-api-combat)
也可以把我对应的文件直接复制过去

## 8.创建自定义异常处理
在 app/Api/Helpers 目录下新建 ExceptionReport.php 文件（对应内容填写我对应的文件里面的）

## 9.捕捉异常

>修改 app/Exceptions 目录下的 Handler.php 文件

```php

<?php

namespace App\Exceptions;
use App\Api\Helpers\ExceptionReport;
use Exception;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{

    public function render($request, Exception $exception)
    {
        //ajax请求我们才捕捉异常
        if ($request->ajax()){
            // 将方法拦截到自己的ExceptionReport
            $reporter = ExceptionReport::make($exception);
            if ($reporter->shouldReturn()){
                return $reporter->report();
            }
            if(env('APP_DEBUG')){
                //开发环境，则显示详细错误信息
                return parent::render($request, $exception);
            }else{
                //线上环境,未知错误，则显示500
                return $reporter->prodReport();
            }
        }
        return parent::render($request, $exception);
    }
}
```

## 10.jwt-auth
>在传统web中，我们一般是使用session来判定一个用户的登陆状态。而在API开发中，我们使用的是token。jwt-token是Laravel开发API用的比较多的。

>jwt-auth的详细介绍分析可以看[JWT超详细分析](https://learnku.com/articles/17883)这篇文章，具体使用可以看[JWT完整使用详解](https://learnku.com/articles/10885/full-use-of-jwt) 这篇文章。

### 10-1. 安装
   ```
   composer require tymon/jwt-auth 1.0.0-rc.5
   如果是Laravel5.5版本，则安装rc.1。如果是Laravel5.6版本，则安装rc.2。我这里安装的最新rc.5版本
   ```
### 10-2.配置
   >配置参考来自使用 [Jwt-Auth 实现 API 用户认证以及无痛刷新访问令牌](https://learnku.com/articles/7264/using-jwt-auth-to-implement-api-user-authentication-and-painless-refresh-access-token)
   
### 10-3. 添加服务提供商 打开 config 目录下的 app.php文件，添加下面代码
   
   ```
   'providers' => [
   
       ...
   
       Tymon\JWTAuth\Providers\LaravelServiceProvider::class,
   ]
   ```
   
### 10-4. 发布配置文件
   
   ```
   php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"

   ```
   >此命令会在 config 目录下生成一个 jwt.php 配置文件，你可以在此进行自定义配置。
   
### 10-5. 生成密钥
      
  ```
  php artisan jwt:secret

  ```
  >此命令会在你的 .env 文件中新增一行 JWT_SECRET=secret。以此来作为加密时使用的秘钥。
  
### 10-6. 配置 Auth guard
  
  >打开 config 目录下的 auth.php文件，修改为下面代码
     
 ```
 'guards' => [
     'web' => [
         'driver' => 'session',
         'provider' => 'users',
     ],
 
     'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
     ],
 ],
 ```
 
 >这样，我们就能让api的用户认证变成使用jwt。
 
### 10-7. 更改 Model
 
 ```
 <?php
 
 namespace App\Models;
 
 use Illuminate\Notifications\Notifiable;
 use Illuminate\Foundation\Auth\User as Authenticatable;
 use Tymon\JWTAuth\Contracts\JWTSubject;
 
 class User extends Authenticatable implements JWTSubject
 {
     use Notifiable;
 
     public function getJWTIdentifier()
     {
         return $this->getKey();
     }
 
     public function getJWTCustomClaims()
     {
         return [];
     }
    ......
 }
 
 ```
 
### 10-8. 配置项详解

  >config目录下的jwt.php文件配置详解
  
```php
  <?php
  
  return [
  
      /*
      |--------------------------------------------------------------------------
      | JWT Authentication Secret
      |--------------------------------------------------------------------------
      |
      | 用于加密生成 token 的 secret
      |
      */
  
      'secret' => env('JWT_SECRET'),
  
      /*
      |--------------------------------------------------------------------------
      | JWT Authentication Keys
      |--------------------------------------------------------------------------
      |
      | 如果你在 .env 文件中定义了 JWT_SECRET 的随机字符串
      | 那么 jwt 将会使用 对称算法 来生成 token
      | 如果你没有定有，那么jwt 将会使用如下配置的公钥和私钥来生成 token
      |
      */
  
      'keys' => [
  
          /*
          |--------------------------------------------------------------------------
          | Public Key
          |--------------------------------------------------------------------------
          |
          | 公钥
          |
          */
  
          'public' => env('JWT_PUBLIC_KEY'),
  
          /*
          |--------------------------------------------------------------------------
          | Private Key
          |--------------------------------------------------------------------------
          |
          | 私钥
          |
          */
  
          'private' => env('JWT_PRIVATE_KEY'),
  
          /*
          |--------------------------------------------------------------------------
          | Passphrase
          |--------------------------------------------------------------------------
          |
          | 私钥的密码。 如果没有设置，可以为 null。
          |
          */
  
          'passphrase' => env('JWT_PASSPHRASE'),
  
      ],
  
      /*
      |--------------------------------------------------------------------------
      | JWT time to live
      |--------------------------------------------------------------------------
      |
      | 指定 access_token 有效的时间长度（以分钟为单位），默认为1小时，您也可以将其设置为空，以产生永不过期的标记
      |
      */
  
      'ttl' => env('JWT_TTL', 60),
  
      /*
      |--------------------------------------------------------------------------
      | Refresh time to live
      |--------------------------------------------------------------------------
      |
      | 指定 access_token 可刷新的时间长度（以分钟为单位）。默认的时间为 2 周。
      | 大概意思就是如果用户有一个 access_token，那么他可以带着他的 access_token 
      | 过来领取新的 access_token，直到 2 周的时间后，他便无法继续刷新了，需要重新登录。
      |
      */
  
      'refresh_ttl' => env('JWT_REFRESH_TTL', 20160),
  
      /*
      |--------------------------------------------------------------------------
      | JWT hashing algorithm
      |--------------------------------------------------------------------------
      |
      | 指定将用于对令牌进行签名的散列算法。
      |
      */
  
      'algo' => env('JWT_ALGO', 'HS256'),
  
      /*
      |--------------------------------------------------------------------------
      | Required Claims
      |--------------------------------------------------------------------------
      |
      | 指定必须存在于任何令牌中的声明。
      | 
      |
      */
  
      'required_claims' => [
          'iss',
          'iat',
          'exp',
          'nbf',
          'sub',
          'jti',
      ],
  
      /*
      |--------------------------------------------------------------------------
      | Persistent Claims
      |--------------------------------------------------------------------------
      |
      | 指定在刷新令牌时要保留的声明密钥。
      |
      */
  
      'persistent_claims' => [
          // 'foo',
          // 'bar',
      ],
  
      /*
      |--------------------------------------------------------------------------
      | Blacklist Enabled
      |--------------------------------------------------------------------------
      |
      | 为了使令牌无效，您必须启用黑名单。
      | 如果您不想或不需要此功能，请将其设置为 false。
      |
      */
  
      'blacklist_enabled' => env('JWT_BLACKLIST_ENABLED', true),
  
      /*
      | -------------------------------------------------------------------------
      | Blacklist Grace Period
      | -------------------------------------------------------------------------
      |
      | 当多个并发请求使用相同的JWT进行时，
      | 由于 access_token 的刷新 ，其中一些可能会失败
      | 以秒为单位设置请求时间以防止并发的请求失败。
      |
      */
  
      'blacklist_grace_period' => env('JWT_BLACKLIST_GRACE_PERIOD', 0),
  
      /*
      |--------------------------------------------------------------------------
      | Providers
      |--------------------------------------------------------------------------
      |
      | 指定整个包中使用的各种提供程序。
      |
      */
  
      'providers' => [
  
          /*
          |--------------------------------------------------------------------------
          | JWT Provider
          |--------------------------------------------------------------------------
          |
          | 指定用于创建和解码令牌的提供程序。
          |
          */
  
          'jwt' => Tymon\JWTAuth\Providers\JWT\Namshi::class,
  
          /*
          |--------------------------------------------------------------------------
          | Authentication Provider
          |--------------------------------------------------------------------------
          |
          | 指定用于对用户进行身份验证的提供程序。
          |
          */
  
          'auth' => Tymon\JWTAuth\Providers\Auth\Illuminate::class,
  
          /*
          |--------------------------------------------------------------------------
          | Storage Provider
          |--------------------------------------------------------------------------
          |
          | 指定用于在黑名单中存储标记的提供程序。
          |
          */
  
          'storage' => Tymon\JWTAuth\Providers\Storage\Illuminate::class,
  
      ],
  
  ];
  
 ```


## 11. 自动刷新用户认证
### 11-1.自定义认证中间件
   ```
   
   php artisan make:middleware Api/RefreshTokenMiddleware
   
   ```
### 11-2. 打开 app/Http/Middleware/Api 目录下的 RefreshTokenMiddleware.php 文件，填写以下内容  
```php
  <?php
  
  namespace App\Http\Middleware\Api;
  
  use Auth;
  use Closure;
  use Tymon\JWTAuth\Exceptions\JWTException;
  use Tymon\JWTAuth\Facades\JWTAuth;
  use Tymon\JWTAuth\Http\Middleware\BaseMiddleware;
  use Tymon\JWTAuth\Exceptions\TokenExpiredException;
  use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;
  
  // 注意，我们要继承的是 jwt 的 BaseMiddleware
  class RefreshTokenMiddleware extends BaseMiddleware
  {
      /**
       * Handle an incoming request.
       *
       * @param  \Illuminate\Http\Request $request
       * @param  \Closure $next
       *
       * @throws \Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException
       *
       * @return mixed
       */
      public function handle($request, Closure $next)
      {
          // 检查此次请求中是否带有 token，如果没有则抛出异常。
          $this->checkForToken($request);
  //         使用 try 包裹，以捕捉 token 过期所抛出的 TokenExpiredException  异常
          try {
              // 检测用户的登录状态，如果正常则通过
              if ($this->auth->parseToken()->authenticate()) {
                  return $next($request);
              }
              throw new UnauthorizedHttpException('jwt-auth', '未登录');
          } catch (TokenExpiredException $exception) {
              // 此处捕获到了 token 过期所抛出的 TokenExpiredException 异常，我们在这里需要做的是刷新该用户的 token 并将它添加到响应头中
              try {
                  // 刷新用户的 token
                  $token = $this->auth->refresh();
                  // 使用一次性登录以保证此次请求的成功
                  Auth::guard('api')->onceUsingId($this->auth->manager()->getPayloadFactory()->buildClaimsCollection()->toPlainArray()['sub']);
              } catch (JWTException $exception) {
                  // 如果捕获到此异常，即代表 refresh 也过期了，用户无法刷新令牌，需要重新登录。
                  throw new UnauthorizedHttpException('jwt-auth', $exception->getMessage());
              }
          }
  
          // 在响应头中返回新的 token
          return $this->setAuthenticationHeader($next($request), $token);
      }
  }
  ```
### 11-3.增加中间件别名
  
  ```
  protected $routeMiddleware = [
      ......
      'api.refresh'=>\App\Http\Middleware\Api\RefreshTokenMiddleware::class,
  ];
  ```
  
### 11-4.单一设备登陆
>同一时间只允许登录唯一一台设备。例如设备 A 中用户如果已经登录，那么使用设备 B 登录同一账户，设备 A 就无法继续使用了。

>我们在登陆，token过期自动更换的时候，都会产生一个新的token。
 我们将token都存到表中的last_token字段。在登陆接口，获取到last_token里的值，将其加入黑名单。
 这样，只要我们无论在哪里登陆，之前的token一定会被拉黑失效，必须重新登陆，我们的目的也就达到了。
 
### 11-5.修改 app\Http\Controllers\Api\AdminController.php 中的 login方法，在登陆的时候，拉黑上一个token。
   
   ```php
    //用户登录
       public function login(Request $request)
       {
           $token = Auth::guard('api')->attempt(['name' => $request->name, 'password' => $request->password]);
           if ($token) {
               //如果登陆，先检查原先是否有存token，有的话先失效，然后再存入最新的token
               $user = Auth::guard('api')->user();
               if ($user['last_token']) {
                   try{
                       Auth::guard('api')->setToken($user['last_token'])->invalidate();
                   }catch (TokenExpiredException $e){
                       //因为让一个过期的token再失效，会抛出异常，所以我们捕捉异常，不需要做任何处理
                   }
               }
               $user->last_token = $token;
               $user->save();  
               return $this->setStatusCode(201)->success(['token' => 'bearer ' . $token]);
           }
           return $this->failed('账号或密码错误', 400);
       }
       
   ```
### 11-6. 再修改中间件app/Http/Middleware/Api/RefreshAdminTokenMiddleware.php ，更新的token加到last_token
   
   ```
   ......
   
   try {
               // 检测用户的登录状态，如果正常则通过
               if ($this->auth->parseToken()->authenticate()) {
                   return $next($request);
               }
               throw new UnauthorizedHttpException('jwt-auth', '未登录');
           } catch (TokenExpiredException $exception) {
               // 3. 此处捕获到了 token 过期所抛出的 TokenExpiredException 异常，我们在这里需要做的是刷新该用户的 token 并将它添加到响应头中
               try {
                   // 刷新用户的 token
                   $token = $this->auth->refresh();
                   // 使用一次性登录以保证此次请求的成功
                   Auth::onceUsingId($this->auth->manager()->getPayloadFactory()->buildClaimsCollection()->toPlainArray()['sub']);
                   //刷新了token，将token存入数据库
                   $user = Auth::user();
                   $user->last_token = $token;
                   $user->save();
               } catch (JWTException $exception) {
                   // 如果捕获到此异常，即代表 refresh 也过期了，用户无法刷新令牌，需要重新登录。
                   throw new UnauthorizedHttpException('jwt-auth', $exception->getMessage());
               }
           }
           
     ......
   ```
   
## 12.horizon管理异步队列

```

composer require laravel/horizon


php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

composer require predis/predis
```
### 12-1.修改队列驱动
  >在.env配置里面 修改 
QUEUE_CONNECTION=redis
并加上REDIS_CLIENT=predis

### 12-2.编写任务类
  ```
  php artisan make:job Api/SaveLastTokenJob

```
### 12-3.打开 app/Jobs/Api/SaveLastTokenJob.php 文件 ，填写以下内容
  
  ```php
  <?php
  
  namespace App\Jobs\Api;
  
  use Illuminate\Bus\Queueable;
  use Illuminate\Queue\SerializesModels;
  use Illuminate\Queue\InteractsWithQueue;
  use Illuminate\Contracts\Queue\ShouldQueue;
  use Illuminate\Foundation\Bus\Dispatchable;
  
  class SaveLastTokenJob implements ShouldQueue
  {
      use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
      protected $model;
      protected $token;
  
      /**
       * SaveLastTokenJob constructor.
       * @param $model
       * @param $token
       */
  
      public function __construct($model,$token)
      {
          //
          $this->model=$model;
          $this->token=$token;
      }
  
      /**
       * Execute the job.
       *
       * @return void
       */
      public function handle()
      {
          $user=$this->model;
          $user->last_token=$this->token;
          $user->save();
      }
  }
  
  ```
  
### 12-4. 使用任务类
    
   ```
   SaveLastTokenJob::dispatch($user,$token);

   ```
### 12-5. 运行Horizon
   
   ```
   php artisan horizon
   
   ```
## 13.Supervisor守护进程 （可参考我的文章）[Centos-7-配置-supervisor-遇到的坑](https://renzhifan.github.io/tool/Centos-7-%E9%85%8D%E7%BD%AE-supervisor-%E9%81%87%E5%88%B0%E7%9A%84%E5%9D%91/)
