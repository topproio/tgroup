---
title: PHP框架开发规范
date: 2019-03-27 18:49:54
tags: 开发规范
---

## 1. 选择：

在没有用户要求及特殊性能要求情况下，我们首先选择Lavarel 框架，版本5.5.X 及以上。如果有极限性能要求，我们选择 Phalcon 作为首选的开发框架，作为C的框架，性能足够强悍，并经过了极客学员核心coreapi的项目检验。

## 2. lavarel 框架使用规范：

### 2.1 目录结构 - 以Laravel5.6为例

**=> App目录**
app 目录包含了应用的核心代码，你为应用编写的代码绝大多数也会放到这里。（注意不是框架的核心代码，框架的核心代码在 /vendor/laravel/framework 里面）。

**=> Bootstrap目录**
bootstrap 目录包含了少许文件，用于框架的启动和自动载入配置，还有一个 cache 文件夹，里面包含了框架为提升性能所生成的文件，如路由和服务缓存文件；

**=> Config目录**
config 目录包含了应用所有的配置文件，建议通读一遍这些配置文件以便熟悉 Laravel 所有默认配置项；

**=> Database目录**
database 目录包含了数据库迁移文件及填充文件，如果有使用 SQLite 的话，你还可以将其作为 SQLite 数据库存放目录；

**=> Public目录**
public 目录包含了应用入口文件 index.php 和前端资源文件（图片、JavaScript、CSS等），该目录也是 Apache 或 Nginx 等 Web 服务器所指向的应用根目录，这样做的好处是隔离了应用核心文件直接暴露于 Web 根目录之下，如果权限系统没做好或服务器配置有漏洞的话，很可能导致应用敏感文件被黑客窃取，进而对网站安全造成威胁；

**=> Resources目录**
resources 目录包含了应用视图文件和未编译的原生前端资源文件（LESS、SASS、JavaScript），以及本地化语言文件；

**=> Routes目录**
routes 目录包含了应用定义的所有路由。Laravel 默认提供了四个路由文件用于给不同的入口使用：web.php、api.php、 console.php 和 channels.php。
web.php 文件包含的路由都位于 RouteServiceProvider 所定义的 web 中间件组约束之内，因而支持 Session、CSRF 保护以及 Cookie 加密功能，如果应用无需提供无状态的、RESTful 风格的 API，那么路由基本上都要定义在 web.php 文件中。
api.php 文件包含的路由位于 api 中间件组约束之内，支持频率限制功能，这些路由是无状态的，所以请求通过这些路由进入应用需要通过 token 进行认证并且不能访问 Session 状态。
console.php 文件用于定义所有基于闭包的控制台命令，每个闭包都被绑定到一个控制台命令并且允许与命令行 IO 方法进行交互，尽管这个文件并不定义 HTTP 路由，但是它定义了基于控制台的应用入口（路由）。
channels.php 文件用于注册应用支持的所有事件广播频道。

**=> Storage目录**
storage 目录包含了编译后的 Blade 模板、基于文件的 Session、文件缓存，以及其它由框架生成的文件，该目录被细分为成 app、framework 和 logs 子目录，app 目录用于存放应用生成的文件，framework 目录用于存放框架生成的文件和缓存，最后，logs 目录存放的是应用的日志文件。
storage/app/public 目录用于存储用户生成的文件，比如可以被公开访问的用户头像，要达到被 Web 用户访问的目的，你还需要在 public （应用根目录下的 public 目录）目录下生成一个软连接 storage 指向这个目录。你可以通过 php artisan storage:link 命令生成这个软链接。

**=> Tests目录**
tests 目录包含自动化测试文件，其中默认已经提供了一个开箱即用的[PHPUnit](https://phpunit.de/) 示例；每一个测试类都要以 Test 开头，你可以通过 phpunit 或 php vendor/bin/phpunit 命令来运行测试。

**=> Vendor目录**
vendor 目录包含了应用所有通过 [Composer](https://getcomposer.org/) 加载的依赖。

### 2.2 特别说明：关于Models

Laravel并没有指定的Models目录，是因为模型层对不同的人有不同的定义。有些开发者认为应用的模型指的是业务逻辑，另外一些人则认为模型指的是与关联数据库的交互。

我个人给出的建议是：models 目录用于存放与数据库交互的模型类，而业务逻辑应该放到 services 目录之下。所以建议在生成模型类的时候指定生成到 app/Models 目录下：
如下面的例子所示：
```
php artisan make:model Models/Test
```

### 2.3 生命周期

**第一件事**
Laravel 应用的所有请求入口都是 public/index.php 文件，所有请求都会被 web 服务器（Apache/Nginx）导向这个文件。 index.php 文件包含的代码并不多，但是，这里是加载框架其它部分的起点。
index.php 文件载入 Composer 生成的自动加载设置，然后从 bootstrap/app.php 脚本获取 Laravel 应用实例，Laravel 的第一个动作就是创建[服务容器](http://laravelacademy.org/post/8695.html)实例。

**HTTP/Console 内核**
接下来，请求被发送到 HTTP 内核或 Console 内核（分别用于处理 Web 请求和 Artisan 命令），这取决于进入应用的请求类型。这两个内核是所有请求都要经过的中央处理器，现在，就让我们聚焦在位于 app/Http/Kernel.php 的 HTTP 内核。
HTTP 内核继承自 Illuminate\Foundation\Http\Kernel 类，该类定义了一个 bootstrappers 数组，这个数组中的类在请求被执行前运行，这些 bootstrappers 配置了错误处理、日志、[检测应用环境](http://laravelacademy.org/post/8650.html#toc_5)以及其它在请求被处理前需要执行的任务。
HTTP 内核还定义了一系列所有请求在处理前需要经过的 HTTP [中间件](http://laravelacademy.org/post/7812.html)，这些中间件处理 [HTTP 会话](http://laravelacademy.org/post/7954.html)的读写、判断应用是否处于维护模式、验证 [CSRF 令牌](http://laravelacademy.org/post/7820.html)等等。
HTTP 内核的 handle 方法签名相当简单：获取一个 Request，返回一个 Response，可以把该内核想象作一个代表整个应用的大黑盒子，输入 HTTP 请求，返回 HTTP 响应。

**服务提供者**
内核启动过程中最重要的动作之一就是为应用载入[服务提供者](http://laravelacademy.org/post/8697.html)，应用的所有服务提供者都被配置在 config/app.php配置文件的 providers 数组中。首先，所有提供者的 register 方法被调用，然后，所有提供者被注册之后，boot 方法被调用。
服务提供者负责启动框架的所有各种各样的组件，比如数据库、队列、验证器，以及路由组件等，正是因为他们启动并配置了框架提供的所有特性，所以服务提供者是整个 Laravel 启动过程中最重要的部分。

**分发请求**
一旦应用被启动并且所有的服务提供者被注册，Request 将会被交给路由器进行分发，路由器将会分发请求到路由或控制器，同时运行所有路由指定的中间件。

### 2.3 路由规则

配置目录：routes目录。

1）基本路由：
```
// 1、get路由
// 访问domain/hello，指向hello模板
Route::get('/hello', function () {
    return view('hello');
});

// 2、post路由
// 访问domain/post，指向hello模板
Route::post('/post', function () {
    return view('hello');
});

// 3、混合路由，get和post都可以。访问[域名/test]即可请求 
Route::match(['get','post'], 'test', function () {
    return '哈哈哈哈哈'; 
}); 

// 4、任意路由，所有请求方式都可以。[域名/hello]即可访问 
Route::any('/hello',function(){ 
    return "Hello Laravel!";
});

```

2）路由关联控制器：
```
// 访问domain/member/info，指向Member控制器下的info方法
Route::get('/member/info','MemberController@info');
```

3）资源路由（常用）：
```
Route::resource('users', 'UsersController');
```

以上资源路由相当于：
```
Route::get('/users', 'UsersController@index')->name('users.index');
Route::get('/users/{user}', 'UsersController@show')->name('users.show');
Route::get('/users/create', 'UsersController@create')->name('users.create');
Route::post('/users', 'UsersController@store')->name('users.store');
Route::get('/users/{user}/edit', 'UsersController@edit')->name('users.edit');
Route::patch('/users/{user}', 'UsersController@update')->name('users.update');
Route::delete('/users/{user}', 'UsersController@destroy')->name('users.destroy');

```

> 我们一般采用资源路由。

## 3 数据库操作

### 3.1 模型：

定义：models 我们认为它的作用应该是与数据库交互，并不做业务逻辑的处理。所以，我们的项目中建议把业务逻辑新建一个services进行处理。
模型设计原则一：Eloquent ORM每个类对应一个数据表名，模型类应与数据表名称相同，数据表名称采用小写下划线，数据库类名采用大写驼峰法。
如表名：user，则数据库类名称应该叫做User。
当表名出现下划线，如user_type，那么类名应该叫做UserType即可。Eloquent ORM模型会自动对应上去。
模型设计原则二：模型的名称应与数据表名称一致，并且主键都设置为"id"，这样无需在模型里面设置主键和表名了

### 3.2 数据库操作：

说明：Laravel里面可以使用原生查询、查询构造器查询和Eloquent ORM查询，这里我们只介绍简单介绍查询构造器和Eloquent ORM的基本操作：

3.2.1 查询构造器的使用：
```
// 增 - 成功返回true
$bool = DB::table('students')->insert([
    'name' => 'linfeng',
    'age' => '22',
]);
// 更新数据
$bool = DB::table('students')->where(['id'=>2])->update([
    'age' => 14,
]);
// 删除
DB::table('students')->where(['id'=>10]);
// get() - 查询数据
$students = DB::table('students')->get();

// first() - 查询单条数据
$students = DB::table('students')->orderBy('id','desc')->first();
```

3.2.2 Eloquent ORM：
```
//all()
$students=Student::all();
//find();
$student=Student::find(1001);
//findOrFail()
$student=Student::findOrFail(1008);

//查询构造器在ORM中的使用：
$students=Student::get();
//first()
$students = Student::where('id','>','1001')
->orderBy('age','desc')
->first();
//chunk()
Student::chunk(2,function ($students){
    var_dump($students);
});
//查询构造器中的聚合函数：
//count
$num=Student::count();
//max()
$max=Student::where('id','>',1001)->max('age');
```

> 请注意，查询构造器返回的是一个数组，而ORM返回的则是一个对象。更多的使用方法请查看官方手册。

### 3.3 关于数据库的命名

为提升开发效率，配合 Laravel 的 ORM，数据表的名称采用复数：
例子如下：
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    //
}
```
—— 请注意，以上的例子并没有指定连接到哪张表，但是在默认的情况下，会连接到数据库的 users表。
如果是 UserType 呢？ORM 会自动连接到 user_types表格

当然，某些特殊情况下（我也不知道是什么特殊情况），可能需要通过指定连接的是哪张表格。如下：
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    //
    protected $table = 'user_type';
}
```

## 4 一些概念介绍

### 4.1 服务容器

Laravel 服务容器是一个用于管理类依赖和执行依赖注入的强大工具。依赖注入听上去很花哨，其实质是通过构造函数或者某些情况下通过 setter 方法将类依赖注入到类中。
文档：[https://laravelacademy.org/post/8695.html](https://laravelacademy.org/post/8695.html)

### 4.2 服务提供者
服务提供者是 Laravel 应用启动的中心，你自己的应用以及所有 Laravel 的核心服务都是通过服务提供者启动。
但是，我们所谓的“启动”指的是什么？通常，这意味着注册服务，包括注册服务容器绑定、事件监听器、中间件甚至路由。服务提供者是应用配置的中心。
如果你打开 Laravel 自带的 config/app.php 文件，将会看到一个 providers 数组，这里就是应用所要加载的所有服务提供者类，当然，其中很多是延迟加载的，也就是说不是每次请求都会被加载，只有真的用到它们的时候才会加载。
文档：[https://laravelacademy.org/post/8697.html](https://laravelacademy.org/post/8697.html)

### 4.3 门面Facades
门面为应用[服务容器](http://laravelacademy.org/post/8695.html)中的绑定类提供了一个“静态”接口。Laravel 内置了很多门面，你可能在不知道的情况下正在使用它们。Laravel 的门面作为服务容器中底层类的“静态代理”，相比于传统静态方法，在维护时能够提供更加易于测试、更加灵活、简明优雅的语法。
文档：[https://laravelacademy.org/post/8708.html](https://laravelacademy.org/post/8708.html)

### 4.4 契约
Laravel 中的契约是指框架提供的一系列定义核心服务的接口。 例如，Illuminate\Contracts\Queue\Queue 契约定义了队列任务需要实现的方法，Illuminate\Contracts\Mail\Mailer 契约定义了发送邮件所需要实现的方法。
每一个契约都有框架提供的相应实现。例如，Laravel 为队列提供了多个驱动的实现，邮件则由 [SwiftMailer](http://swiftmailer.org/) 驱动实现 。
所有 Laravel 契约都有其[对应的GitHub库](https://github.com/illuminate/contracts)，这为所有有效的契约提供了快速入门指南，同时也可以作为独立、解耦的包被包开发者使用。
[https://laravelacademy.org/post/8710.html](https://laravelacademy.org/post/8710.html)

## 5 一些常用命令：
1）composer安装laravel：
```
// 安装最新稳定版
composer create-project --prefer-dist laravel/laravel laravel
// 指定安装5.3.*版本
composer create-project laravel/laravel --prefer-dist laravel "5.6.*" 
```
2）显示版本：
```
php artisan –version
```
3）创建控制器 - 在 app/controllers 目录下生成一个名为 控制器名.php 的控制器文件
```
php artisan controller:make 控制器名
```
4）创建模型：
```
// 创建模型并创建新迁移
php artisan make:model User --migration 
```
5）清除应用程序缓存：
```
php artisan cache:clear
```
6）开启Auth用户功能：
```
php artisan make:auth
```

## 6 RESTful风格和常用状态码：

### 6.1 路由地址定义：
| 方法 | 请求路径   | 方法/动作 | 路由名称   | 
|:----|:----|
| GET | /users   | index | users.index   | 
| GET | /users/create   | create | users.create   | 
| POST | /users   | store | users.store   | 
| GET | /users/{id}   | show | users.show   | 
| GET | /users/{id}/edit   | edit | users.edit   | 
| PUT/PATCH | /users/{id}   | update | users.update   | 
| DELETE | /users/{id}   | destroy | users.destroy   | 

### 6.2 常用状态码
**200 OK - 对成功的 GET、PUT、PATCH 或 DELETE 操作进行响应。也可以被用在不创建新资源的 **
POST 操作上
**201 Created - 对创建新资源的 POST 操作进行响应。应该带着指向新资源地址的 Location 头**
202 Accepted - 服务器接受了请求，但是还未处理，响应中应该包含相应的指示信息，告诉客户端该去哪里查询关于本次请求的信息
204 No Content - 对不会返回响应体的成功请求进行响应（比如 DELETE 请求）
304 Not Modified - HTTP缓存header生效的时候用
400 Bad Request - 请求异常，比如请求中的body无法解析
**401 Unauthorized - 没有进行认证或者认证非法**
**403 Forbidden - 服务器已经理解请求，但是拒绝执行它**
**404 Not Found - 请求一个不存在的资源**
**405 Method Not Allowed - 所请求的 HTTP 方法不允许当前认证用户访问**
410 Gone - 表示当前请求的资源不再可用。当调用老版本 API 的时候很有用
415 Unsupported Media Type - 如果请求中的内容类型是错误的
422 Unprocessable Entity - 用来表示校验错误
429 Too Many Requests - 由于请求频次达到上限而被拒绝访问

## 7 一些开发中书写规范

### 7.1 编码规范

1）目录命名：文件夹名称**必须**符合CamelCase式的大写字母开头驼峰命名规范。
2）文件：纯PHP代码文件**必须**省略最后的 ?> 结束标签。
3）行的长度**一定**限制在140个字符以内。
4）非空行后**一定**不能有多余的空格符。
5）每行**一定**不能存在多于一条语句。
6）适当空行**可以**使得阅读代码更加方便以及有助于代码的分块（但注意不能滥用空行）。
7）代码**必须**使用4个空格符的缩进，**一定不能**用 tab键 。
8）PHP所有**关键字****必须**全部小写。常量 **true**、**false**和 **null****必须**全部小写。
9）**namespace **声明后**必须**插入一个空白行。所有**use****必须**在**namespace **后声明。每条**use**声明语句**必须**只有一个**use**关键词。**use **声明语句块后**必须**要有一个空白行。
```
<?php
namespace VendorgiPackage;

use FooClass;
use BarClass as Bar;
use OtherVendorgiOtherPackage\BazClass;

class ClassName
{
    // constants, properties, methods
}

```

### 7.2 类、属性和方法

类的命名**必须**遵循大**写字母**开头的驼峰式命名规范：
```
class UserType 
{
  
}
```

类的常量中所有字母都**必须**大写，词间以**下划线分隔**。
```
USER_TYPE
```

类的属性命名**必须**遵循**小写字母**开头的驼峰式命名规范 ($camelCase)。**必须**对所有属性设置访问控制（如，**public**，**protect**，**private**）。
```
private $name;
```

方法名称**必须**符合 camelCase() 式的**小写字母**开头驼峰命名规范。所有方法都**必须**设置访问控制（如，**public**，**protect**，**private**）。
```
  public function fooBarBaz($storeName, $storeId, array $info = [])
    {
        // method body
    }
```

方法参数名称**必须**符合 camelCase 式的**小写字母**开头驼峰命名规范
```
public function foo($storeName, $storeId, BazClass $bazObj, array $info = [])
{
    // method body
}
```

### 7.3 类注释

1）注释开始 /** **不可以** /*，结束 */ 不可以 **/。
2）第二行开始描述，描述后一空行。
3）注解内容对齐，注解之间**不可**有空行。
4）星号和注释内容中间**必须**是一个空格。
```
/**
 * 我是类描述信息哦！
 * 
 * @author  Author
 * @since   2015年1月12日
 * @version 1.0
 */
```

### 7.4 属性注释

1）注释开始 /** **不可以** /*，结束 */ **不可以** **/。
2）星号和注释内容中间**必须**是一个空格。
3）使用var注解并注明类型。
4）注解基本类型包括int、sting、array、boolea、具体类名称。

```
<?php
class Cache
{
    /**
     * 缓存的键值
     * @var string
     */
	public static $cacheKey = '';
	
    /**
     * 缓存的键值
     * @var string
     */
	public static $cacheTime = 60;
	
    /**
     * 缓存的对象
     * @var \CacheServer
     */
    public static $cacheObj = null;
}
```

