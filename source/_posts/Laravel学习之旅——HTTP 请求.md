---
title: Laravel学习之旅——HTTP 请求
date: 2017-04-01 13:48:00
tags: [PHP, Laravel]
categories: [Laravel]
toc: true
cover: '/images/categories/laravel.jpeg'
---

# 访问请求

通过依赖注入获取当前HTTP请求实例，应该在控制器的构造函数或方法中对 ` Illuminate\Http\Request `类进行类型提示，当前请求实例会被服务容器自动注入：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    /**
     * 存储新用户
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name=$request->input('name'); 
        
        //
    }
}
```

如果你的控制器方法还期望获取路由参数输入，只需要将路由参数置于其它依赖之后即可，例如，如果你的路由定义如下：

```php
Route::put('user/{id}','UserController@update');
```

你仍然可以对 `Illuminate\Http\Request` 进行类型提示并通过如下方式定义控制器方法来访问路由参数：

```php
<?php
    
namespace App\Http\Controllers;
    
use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    /**
     * 更新指定用户
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
     public function update(Request $request,$id)
    { 
        //
    }
}
```

## 基本请求信息

`Illuminate\Http\Request` 实例提供了多个方法来检测应用的HTTP请求， `Laravel` 的 `Illuminate\Http\Request` 继承自 `Symfony\Component\HttpFoundation\Request` 类，这里列出了一些该类中的有用方法：

### 获取请求URI

path方法将会返回请求的URI，因此，如果进入的请求路径是 `http://domain.com/foo/bar` ，则 `path` 方法将会返回 `foo/bar`：

```php
$uri=$request->path();
```

`is` 方法允许你验证进入的请求是否与给定模式匹配。使用该方法时可以使用 `*` 通配符：

```php
if($request->is('admin/*')){ 
    //
}
```

想要获取完整的URL，而不仅仅是路径信息，可以使用请求实例中的 `url` 方法：

```php
$url=$request->url();
```

### 获取请求方法

`method` 方法将会返回请求的HTTP请求方式。你还可以使用 `isMethod` 方法来验证HTTP请求方式是否匹配给定字符串：

```php
$method=$request->method();
if($request->isMethod('post')){ 
    //
}
```

## PSR-7 请求

`PSR-7` 标准指定了HTTP消息接口，包括请求和响应。如果你想要获取PSR-7请求实例，首先需要安装一些库，Laravel使用Symfony HTTP Message Bridge组件将典型的Laravel请求和响应转化为PSR-7兼容的实现：

```bash
composer require symfony/psr-http-message-bridge
composer require zendframework/zend-diactoros
```

安装完这些库之后，你只需要在路由或控制器中通过对请求类型进行类型提示就可以获取PSR-7请求：

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    //
});
```

如果从路由或控制器返回的是PSR-7响应实例，则其将会自动转化为Laravel响应实例并显示出来。

# 获取输入

### 获取输入值

使用一些简单的方法，就可以从 `Illuminate\Http\Request` 实例中访问用户输入。你不需要担心请求所使用的HTTP请求方法，因为对所有请求方式的输入访问接口都是一致的：

```php
$name = $request->input('name');
```

你还可以传递一个默认值作为第二个参数给input方法，如果请求输入值在当前请求未出现时该值将会被返回：

```php
$name = $request->input('name', 'Sally');
```

处理表单数组输入时，可以使用”.”来访问数组：

```php
$input = $request->input('products.0.name');
```

### 判断输入值是否出现

判断值是否在请求中出现，可以使用 `has` 方法，如果值出现过了且不为空， `has` 方法返回 `true`：

```php
if ($request->has('name')) {
    //
}
```

### 获取所有输入数据

你还可以通过all方法获取所有输入数据：

```php
$input = $request->all();
```

### 获取输入的部分数据

如果你需要取出输入数据的子集，可以使用 `only`或 `except` 方法，这两个方法都接收一个数组作为唯一参数：

```php
$input = $request->only('username', 'password');
$input = $request->except('credit_card');
```

## 上一次请求输入

Laravel允许你在两次请求之间保存输入数据，这个特性在检测校验数据失败后需要重新填充表单数据时很有用，但如果你使用的是Laravel内置的验证服务，则不需要手动使用这些方法，因为一些Laravel内置的校验设置会自动调用它们。

### 将输入存储到一次性 `Session`

`Illuminate\Http\Request` 实例的 `flash` 方法会将当前输入存放到一次性 `session` （所谓的一次性指的是从session中取出数据中，对应数据会从session中销毁）中，这样在下一次请求时数据依然有效：

```php
$request->flash();
```

你还可以使用 `flashOnly` 和 `flashExcept` 方法将输入数据子集存放到session中：

```php
$request->flashOnly('username', 'email');
$request->flashExcept('password');
```

### 将输入存储到一次性Session然后重定向

如果你经常需要一次性存储输入并重定向到前一页，可以简单使用 `withInput` 方法来将输入数据链接到 `redirect` 后面：

```php
return redirect('form')->withInput();
return redirect('form')->withInput($request->except('password'));
```

###  取出上次请求数据

要从session中取出上次请求的输入数据，可以使用Request实例的old方法。old方法提供了便利的方式从session中取出一次性数据：

```php
$username = $request->old('username');
```

Laravel还提供了一个全局的帮助函数 `old` ，如果你是在 `Blade` 模板中显示老数据，使用帮助函数 `old` 更方便：

```php
{{ old('username') }}
```

## Cookies

### 从请求中取出Cookies

Laravel框架创建的所有cookies都经过加密并使用一个认证码进行签名，这意味着如果客户端修改了它们则需要对其进行有效性验证。我们使用 `Illuminate\Http\Request` 实例的cookie方法从请求中获取cookie的值：

```php
$value = $request->cookie('name');
```
### 新增Cookie

Laravel提供了一个全局的帮助函数cookie作为一个简单工厂来生成新的
`Symfony\Component\HttpFoundation\Cookie`实例，新增的 cookies 通过 `withCookie` 方法被附加到 `Illuminate\Http\Response` 实例：

```php
$response = new Illuminate\Http\Response('Hello World');
$response->withCookie(cookie('name', 'value', $minutes));
return $response;
```

想要创建一个长期有效的cookie，可以使用cookie工厂的 `forever` 方法：

```php
$response->withCookie(cookie()->forever('name', 'value'));
```

## 文件上传

### 获取上传的文件

可以使用 `Illuminate\Http\Request` 实例的 `file` 方法来访问上传文件，该方法返回的对象是 `Symfony\Component\HttpFoundation\File\UploadedFile` 类的一个实例，该类继承自PHP标准库中提供与文件交互方法的 `SplFileInfo` 类：

```php 
$file = $request->file('photo');
```

### 验证文件是否存在

使用hasFile方法判断文件在请求中是否存在：

```php
if ($request->hasFile('photo')) {
    //
}
```

### 验证文件是否上传成功

使用 `isValid` 方法判断文件在上传过程中是否出错：

```php
if ($request->file('photo')->isValid()){
    //
}
```

### 保存上传的文件

使用 `move` 方法将上传文件保存到新的路径，该方法将上传文件从临时目录（在PHP配置文件中配置）移动到指定新目录：

```php
$request->file('photo')->move($destinationPath);
$request->file('photo')->move($destinationPath, $fileName);
```

### 其它文件方法

`UploadedFile` 实例中很有很多其它方法，查看该类的[APT](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html)了解更多相关方法。

















