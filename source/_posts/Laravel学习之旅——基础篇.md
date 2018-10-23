---
title: Laravel学习之旅——基础篇（安装、路由、CSRF、404）
date: 2017-04-01 09:31:25
tags: [PHP, Laravel]
toc: true
---

# 安装

## 对运行环境要求

Laravel 框架对系统环境有一些要求。当然，所有这些要求在 Laravel Homestead 虚拟机中都是预装好的：

- PHP >= 5.5.9 - OpenSSL PHP 扩展 - PDO PHP 扩展 - Mbstring PHP 扩展 - Tokenizer PHP 扩展

## 安装 Laravel

### 下载 Laravel 一键安装包

下载地址：http://www.golaravel.com/download/
<br>
如图，我下载的是laravel5.1 LTS版本:
<img src="/images/laravel版本.png" />

# 路由

#### 我们将在`app\Http\routes.php`文件中定义应用程序的大多数路由，该文件由`App\Providers\RouteServiceProvider`类加载。 最基本的Laravel路由只接受一个URI和一个Closure：

```php
Route::get('/', function () {
    return 'Hello World';
});

Route::post('foo/bar', function () {
    return 'Hello World';
});

Route::put('foo/bar', function () {
    //
});

Route::delete('foo/bar', function () {
    //
});
```
    
#### 有时我们可能需要注册一个响应多个HTTP响应的路由。 可以使用 `match` 路由匹配方法：

```php
Route::match(['get', 'post'], '/', function () {
    return 'Hello World';
});
//这样，对于‘/’的get和post请求都会使用这个路由，并返回hello World。
```
    
#### 或者我们也可以注册一个可以响应所有HTTP请求的 `any` 路由方法：

```php
Route::any('foo', function () {
    return 'Hello World';
});
//这样无论是get、post或其他的请求方式，/foo路由都返回Hello world。
```
    
#### 带参数路由请求：

```php
//带一个参数
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

#### 带多个参数路由请求：

```php
//带多个参数
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

#### 有时，我们可能需要指定路由参数，但使该路由参数的存在是可选的。 我们可以通过放置一个`？` 标记在参数名称后面：

```php
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

#### 正则表达式路由匹配
 
```php
//name字段只能是大小写字母
Route::get('user/{name}', function ($name) {
    //
})
->where('name', '[A-Za-z]+');

//id必须为数字
Route::get('user/{id}', function ($id) {
    //
})
->where('id', '[0-9]+');

//id必须为数字，name为小写字母
Route::get('user/{id}/{name}', function ($id, $name) {
    //
})
->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

#### 全局约束

如果希望路由参数始终受到给定正则表达式的约束，则可以使用pattern方法。 您应该在`App\Providers\RouteServiceProvider.php`的`boot`方法中定义这些模式：

```php
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @param  \Illuminate\Routing\Router  $router
 * @return void
 */
public function boot(Router $router)
{
    $router->pattern('id', '[0-9]+');

    parent::boot($router);
}
```

一旦定义了模式，它将自动应用于使用该参数名的所有路由：

```php
//id自动回匹配RouteServiceProvider里面定义的正则规则
Route::get('user/{id}', function ($id) {
   // Only called if {id} is numeric.
});
```

#### 命名路由

通过命名路由，我们可以方便地为特定路由生成URL或重定向。 可以在定义路由时使用`as`数组键为路由指定名称：

```php
Route::get('user/profile', ['as' => 'profile', function () {
    //
}]);
```
    
还可以为控制器操作指定路由名称:

```php
Route::get('user/profile', [
    'as' => 'profile', 'uses' => 'UserController@showProfile'
]);
```

代替在路由数组定义中指定路由名称，我们可以将名称方法链接到路由定义的末尾：

```php
Route::get('user/profile', 'UserController@showProfile')->name('profile');
```

#### 路由组 和 命名路由

如果我们使用路由组，我们可以在路由组属性数组中指定`as`关键字，允许为组内的所有路由设置公共路由名前缀：

```php
Route::group(['as' => 'admin::'], function () {
    Route::get('dashboard', ['as' => 'dashboard', function () {
        // Route named "admin::dashboard"
    }]);
});
```

#### 生成URL到命名路由

为指定路线指定名称后，我们可以在通过路线功能生成网址或重定向时使用路线名称：

```php
$url = route('profile');
$redirect = redirect()->route('profile');
```

如果路由定义参数，我们可以将参数作为第二个参数传递给route方法。 给定的参数将自动插入到URL中：

```php
Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
    //
}]);
$url = route('profile', ['id' => 1]);
```

#### 路由组

路由组允许跨大量路由共享路由属性（例如中间件或命名空间），而无需在每个单独路由上定义这些属性。 共享属性以数组格式指定为`Route :: group`方法的第一个参数。

##### 中间件

要将中间件分配给组中的所有路由，我们可以使用组属性数组中的中间件键。 中间件将按照我们定义此数组的顺序执行：

```php
Route::group(['middleware' => 'auth'], function () {
    Route::get('/', function ()    {
        // Uses Auth Middleware
    });

    Route::get('user/profile', function () {
        // Uses Auth Middleware
    });
});
```

#### 命名空间

路由组的另一个常见用例是将同一个PHP命名空间分配给一组控制器。 您可以在组属性数组中使用`namespace`参数来为组中的所有控制器指定命名空间：

```php
Route::group(['namespace' => 'Admin'], function()
{
    // Controllers Within The "App\Http\Controllers\Admin" Namespace

    Route::group(['namespace' => 'User'], function()
    {
        // Controllers Within The "App\Http\Controllers\Admin\User" Namespace
    });
});
```

记住，默认情况下，`RouteServiceProvider`在命名空间组中包含您的`routes.php`文件，允许您注册控制器路由，而不指定完整的`App\Http\Controllers`命名空间前缀。 因此，我们只需要指定在`App\Http\Controllers`命名空间根目录之后的命名空间部分。

##### 子域路由

路由组也可以用于路由通配符子域。 子域可以像路由URI一样被分配路由参数，允许我们捕获子域的一部分以在路由或控制器中使用。 可以使用组属性数组上的域密钥指定子域：

```php
Route::group(['domain' => '{account}.myapp.com'], function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```
    
    
#### 路由前缀    
    
   前缀组数组属性可以用于使用给定的URI为组中的每个路由加前缀。 例如，您可能希望在组内用`admin`路由URI作为前缀： 
    
```php
Route::group(['prefix' => 'admin'], function () {
     Route::get('users', function ()    {
         // Matches The "/admin/users" URL
     });
});   
```
    
您还可以使用prefix参数为您的分组路由指定通用参数：    
    
```php
Route::group(['prefix' => 'accounts/{account_id}'], function () {
    Route::get('detail', function ($account_id)    {
        // Matches The accounts/{account_id}/detail URL
    });
});
```


# CSRF保护

Laravel可以轻松保护您的应用程序免受跨站点请求伪造。 跨站点请求伪造是一种恶意利用，其中代表已验证用户执行未授权命令。
<br>
Laravel为应用程序管理的每个活动用户会话自动生成CSRF“令牌”。 此令牌用于验证已认证的用户是实际向应用程序发出请求的用户。 要生成包含CSRF令牌的隐藏输入字段_token，可以使用csrf_field帮助函数：

```php
<?php echo csrf_field(); ?>
```


`csrf_field`函数主要放在下面的html中

```html
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```
    
当然，也可以使用Blade模板引擎：

```php
{{ csrf_field() }}
```
    
您不需要在`POST`，`PUT`或`DELETE`请求中手动验证CSRF令牌。 `VerifyCsrfToken` HTTP中间件将验证请求输入中的令牌是否与会话中存储的令牌匹配。

## 从CSRF保护中排除指定URI

有时您可能希望从 CSRF 保护中排除一组 URI。 例如，如果您使用`Stripe`处理付款并使用其 Webhook 系统，则需要从 Laravel 的 CSRF 保护中排除您的 Webhook 处理程序路径。

您可以通过将 URI 添加到`VerifyCsrfToken`中间件的 `$ except` 属性中来排除URI：

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

class VerifyCsrfToken extends BaseVerifier
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
    ];
}
```

# X-CSRF-TOKEN

除了检查CSRF令牌作为POST参数之外，Laravel VerifyCsrfToken中间件还将检查`X-CSRF-TOKEN`请求头。 例如，您可以将令牌存储在“`meta`”标记中：
    
```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

创建元标记后，您可以指示像jQuery这样的库将令牌添加到所有请求标头。 这为您的基于AJAX的应用程序提供了简单，方便的CSRF保护：

```js
$.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
});
```

# X-XSRF-TOKEN

Laravel还将CSRF令牌存储在XSRF-TOKEN cookie中。 您可以使用cookie值设置X-XSRF-TOKEN请求标头。 一些JavaScript框架，如Angular，会自动为您执行此操作。 您不太可能需要手动使用此值。

# #路由模型绑定

Laravel路由模型绑定提供了一种方便的方法来将类实例注入到路由中。 例如，您可以注入与给定`ID`匹配的整个`User`类实例，而不是注入用户的`ID`。

首先，使用路由器的模型方法指定给定参数的类。 您应该在`RouteServiceProvider :: boot`方法中定义模型绑定：

#### Binding A Parameter To A Model

```php
public function boot(Router $router)
{
    parent::boot($router);

    $router->model('user', 'App\User');
}
```

接下来，定义包含`{user}`参数的路由：

```php
$router->get('profile/{user}', function(App\User $user) {
    //
});
```
    
因为我们已经将`{user}`参数绑定到`App\User`模型，所以一个User实例将被注入到路由中。 因此，例如，对`profile/1`的请求将注入具有ID为1的User实例。  
    
注意：如果在数据库中找不到匹配的模型实例，则会自动抛出404异常。
    
如果你想指定你自己的“未找到”行为，传递一个Closure作为模型方法的第三个参数：
    
```php
$router->model('user', 'App\User', function() {
    throw new NotFoundHttpException;
});  
```
    
    
如果你想使用自己的解析逻辑，你应该使用`Route :: bind`方法。 传递给`bind`方法的`Closure`将接收 URI 段的值，并且应该返回要注入到路由中的类的实例：    
    
```php
$router->bind('user', function($value) {
    return App\User::where('name', $value)->first();
});  
```
    
# #Form Method Spoofing

HTML表单不支持`PUT`，`PATCH`或`DELETE`操作。 因此，当定义从 HTML 表单调用的`PUT`，`PATCH`或`DELETE`路由时，您将需要向表单添加隐藏的`_method`字段。 与`_method`字段一起发送的值将用作 HTTP 请求方法：    
    
```html
 <form action="/foo/bar" method="POST">
     <input type="hidden" name="_method" value="PUT">
     <input type="hidden" name="_token" value="{{ csrf_token() }}">
 </form> 
 ```

要生成隐藏的输入字段_method，您还可以使用method_field帮助函数：
    
```php
<?php echo method_field('PUT'); ?>   
```
    
当然，也可以使用Blade模板引擎：   
  
```php
{{ method_field('PUT') }}    
```
    
# #Throwing 404 Errors   

有两种方法可以从路由中手动触发404错误。 首先，您可以使用中止帮助程序。 中止帮助器只是抛出一个带有指定状态码的`Symfony\Component\HttpFoundation\Exception\HttpException`：

```php
abort(404);
```

其次，您可以手动抛出`Symfony\Component\HttpKernel\Exception\NotFoundHttpException`的实例。