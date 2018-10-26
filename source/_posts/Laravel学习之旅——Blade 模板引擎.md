---
title: 【转】Laravel学子之旅——Blade 模板引擎
date: 2017-04-02 11:56:43
tags: [PHP, Laravel]
categories: [Laravel]
toc: true
cover: '/images/categories/laravel.jpeg'
---

#注：MarkDown不支持两个'}'连续出现，所以下文使用'\}'替换！

# 简介

`Blade` 是Laravel提供的一个非常简单、强大的`模板引擎`，不同于其他流行的PHP模板引擎，Blade在`视图`中并不约束你使用PHP原生代码。所有的Blade视图都会被编译成原生PHP代码并缓存起来直到被修改，这意味着对应用的性能而言Blade基本上是零开销。Blade视图文件使用 `.blade.php`文件扩展并存放在 `resources/views` 目录下。

# 模板继承

## 定义布局

使用Blade的两个最大优点是模板`继承`和`切片`，开始之前让我们先看一个例子。首先，我们检测一个“主”页面布局，由于大多数web应用在不同页面中使用同一个布局，可以很方便的将这个布局定义为一个单独的Blade页面：

```html
<!-- 存放在 resources/views/layouts/master.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

正如你所看到的，该文件包含典型的HTML标记，然而，注意 `@section`和 `@yield` 指令，前者正如其名字所暗示的，定义了一个内容的片段，而后者用于显示给定片段的内容。

现在我们已经为应用定义了一个布局，接下来让我们定义继承该布局的子页面吧。

## 扩展布局

定义子页面的时候，可以使用Blade的 `@extends`指令来指定子页面所继承的布局，继承一个Blade布局的视图将会使用 `@section` 指令注入内容到布局的片段中，记住，如上面例子所示，这些片段的内容将会显示在布局中使用 `@yield` 的地方：

```html
<!-- 存放在 resources/views/layouts/child.blade.php -->

@extends('layouts.master')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

在本例中，sidebar片段使用 `@parent` 指令来追加（而非覆盖）内容到布局中sidebar， `@parent` 指令在视图渲染时将会被布局中的内容替换。

当然，和原生PHP视图一样，Blade视图可以通过view方法直接从路由中返回：

```php
Route::get('blade', function () {
    return view('child');
});
```
# 数据显示

可以通过两个花括号包裹变量来显示传递到视图的数据，比如，如果给出如下路由：

```php
Route::get('greeting', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

那么可以通过如下方式显示 `name` 变量的内容：

```php
Hello, {{ $name }}.
```

当然，不限制显示到视图中的变量内容，你还可以输出任何PHP函数，实际上，可以将任何PHP代码放到Blade模板语句中：

```php
The current UNIX timestamp is {{ time() }}.
```

注意：Blade的`{{``}}`语句已经经过PHP的`htmlentities`函数处理以避免XSS攻击。

### Blade & JavaScript框架

由于很多JavaScript框架也是用花括号来表示要显示在浏览器中的表达式，可以使用 `@` 符号来告诉Blade渲染引擎该表达式应该保持原生格式不作改动。比如：

```html
<h1>Laravel</h1>

Hello, @{{ name }}.
```

在本例中，`@`符将会被Blade移除，然而，`{{ name }}`表达式将会保持不变，避免被JavaScript框架渲染。

### 输出存在的数据

有时候你想要输出一个变量，但是不确定该变量是否被设置，我们可以通过如下PHP代码：

```php
{{ isset($name) ? $name : 'Default' }}
```

除了使用三元运算符，Blade还提供了更简单的方式：

```php
{{ $name or 'Default' }}
```

在本例中，如果`$name`变量存在，其值将会显示，否则将会显示“Default”。

### 显示原生数据

默认情况下，Blade的`{{``}}`语句已经通过PHP的`htmlentities`函数处理以避免XSS攻击，如果你不想要数据被处理，可以使用如下语法：

```php
Hello, {!! $name !!}.
```

注意：输出用户提供的内容时要当心，对用户提供的内容总是要使用双花括号包裹以避免直接输出HTML代码。

# 流程控制

除了模板继承和数据显示之外，Blade还为常用的PHP流程控制提供了便利操作，比如条件语句和循环，这些快捷操作提供了一个干净、简单的方式来处理PHP的流程控制，同时保持和PHP相应语句的相似。

## If语句

可以使用`@if`, `@elseif`, `@else`, 和 `@endif`来构造if语句，这些指令函数和PHP的相同：

```php
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

为方便起见，Blade还提供了`@unless`指令：

```php
@unless (Auth::check())
    You are not signed in.
@endunless
```

## 循环

除了条件语句，Blade还提供了简单指令处理PHP支持的循环结构，同样，这些指令函数和PHP的一样：

```php
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

## 包含子视图

Blade的@include指令允许你很简单的在一个视图中包含另一个Blade视图，所有父级视图中变量在被包含的子视图中依然有效：

```html
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

尽管被包含的视图继承所有父视图中的数据，你还可以传递额外参数到被包含的视图：

```php
@include('view.name', ['some' => 'data'])
```

## 注释

Blade还允许你在视图中定义注释，然而，不同于HTML注释，Blade注释并不会包含到HTML中被返回：

```php
{{-- This comment will not be present in the rendered HTML --}}
```

# 服务注入

`@inject`指令可以用于从服务容器中获取服务，传递给`@inject`的第一个参数是服务将要被放置到的变量名，第二个参数是要解析的服务类名或接口名：

```html
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

# 扩展Blade

Blade甚至还允许你自定义指令，可以使用`directive`方法来注册一个指令。当Blade编译器遇到该指令，将会传入参数并调用提供的回调。

下面的例子创建了一个`@datetime($var)`指令：

```php
<?php

namespace App\Providers;

use Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::directive('datetime', function($expression) {
            return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
        });
    }

    /**
     * 在容器中注册绑定.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

正如你所看到的，Laravel的帮助函数`with`被用在该指令中，`with`方法简单返回给定的对象/值，允许方法链。最终该指令生成的PHP代码如下：

```php
<?php echo with($var)->format('m/d/Y H:i'); ?>
```