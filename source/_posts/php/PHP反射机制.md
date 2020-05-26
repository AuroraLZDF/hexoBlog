---
title: PHP反射机制
date: 2020-05-21 14:00:53
categories: [PHP]
tags: [PHP]
---

# [转：PHP反射机制](https://www.cnblogs.com/daxiaohaha/p/11542374.html)

# 简介

就算是类成员定义为 **private** 也可以在外部访问，不用创建类的实例也可以访问类的成员和方法。

PHP自5.0版本以后添加了反射机制，它提供了一套强大的反射API，允许你在PHP运行环境中，访问和使用类、方法、属性、参数和注释等，其功能十分强大，经常用于高扩展的PHP框架，自动加载插件，自动生成文档，甚至可以用来扩展PHP语言。由于它是PHP內建的oop扩展，为语言本身自带的特性，所以不需要额外添加扩展或者配置就可以使用。更多内容见官方文档。

# 反射类型

PHP反射API会基于类，方法，属性，参数等维护相应的反射类，已提供相应的调用API。

| 类型                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| Reflector                  | Reflector 是一个接口，被所有可导出的反射类所实现（implement） |
| Reflection                 | 反射（reflection）类                                         |
| ReflectionClass            | 报告了一个类的有关信息                                       |
| ReflectionClassConstant    | 报告有关类常量的信息                                         |
| ReflectionZendExtension    | 报告Zend扩展的相关信息                                       |
| ReflectionExtension        | 报告了PHP扩展的有关信息                                      |
| ReflectionFunction         | 报告了一个函数的有关信息                                     |
| ReflectionFunctionAbstract | ReflectionFunction 的父类                                    |
| ReflectionMethod           | 报告了一个方法的有关信息                                     |
| ReflectionObject           | 报告了一个对象（object）的相关信息                           |
| ReflectionParameter        | 取回了函数或方法参数的相关信息                               |
| ReflectionProperty         | 报告了类的属性的相关信息                                     |
| ReflectionType             | 获取函数、类方法的参数或者返回值的类型                       |
| ReflectionGenerator        | 获取生成器的信息                                             |
| ReflectionException        | 反射异常类                                                   |

# 访问

假设定义了一个类 User，我们首先需要建立这个类的反射类实例，然后基于这个实例可以访问 User 中的属性或者方法。不管类中定义的成员权限声明是否为 public，都可以获取到。

```php
<?php 
namespace Extend;

use ReflectionClass;
use Exception;

/**
 * Class User
 * @package Extend
 */
class User{
    const ROLE = 'Students';
    public $username = '';
    private $password = '';

    public function __construct($username, $password)
    {
        $this->username = $username;
        $this->password = $password;
    }

    /**
     * 获取用户名
     * @return string
     */
    public function getUsername()
    {
        return $this->username;
    }

    /**
     * 设置用户名
     * @param string $username
     */
    public function setUsername($username)
    {
        $this->username = $username;
    }

    /**
     * 获取密码
     * @return string
     */
    private function getPassword()
    {
        return $this->password;
    }

    /**
     * 设置密码
     * @param string $password
     */
    private function setPassowrd($password)
    {
        $this->password = $password;
    }
}

$class = new ReflectionClass('Extend\User');  // 将类名User作为参数，即可建立User类的反射类
$properties = $class->getProperties();  // 获取User类的所有属性，返回ReflectionProperty的数组
$property = $class->getProperty('password'); // 获取User类的password属性ReflectionProperty
$methods = $class->getMethods();   // 获取User类的所有方法，返回ReflectionMethod数组
$method = $class->getMethod('getUsername');  // 获取User类的getUsername方法的ReflectionMethod
$constants = $class->getConstants();   // 获取所有常量，返回常量定义数组
$constant = $class->getConstant('ROLE');   // 获取ROLE常量
$namespace = $class->getNamespaceName();  // 获取类的命名空间
$comment_class = $class->getDocComment();  // 获取User类的注释文档，即定义在类之前的注释
$comment_method = $class->getMethod('getUsername')->getDocComment();  // 获取User类中getUsername方法的注释文档
```

# 交互

一旦创建了反射类的实例，我们不仅可以通过反射类访问原来类的方法和属性，还能创建原来类的实例或则直接调用类里面的方法。

```php
$class = new ReflectionClass('Extend\User');  // 将类名User作为参数，即可建立User类的反射类
$instance = $class->newInstance('youyou', 1, '***');  // 创建User类的实例

$instance->setUsername('youyou_2');  // 调用User类的实例调用setUsername方法设置用户名
$value = $instance->getUsername();   // 用过User类的实例调用getUsername方法获取用户名
echo $value;echo "\n";   // 输出 youyou_2

$class->getProperty('username')->setValue($instance, 'youyou_3');  // 通过反射类ReflectionProperty设置指定实例的username属性值
$value = $class->getProperty('username')->getValue($instance);  // 通过反射类ReflectionProperty获取username的属性值
echo $value;echo "\n";   // 输出 youyou_3

$class->getMethod('setUsername')->invoke($instance, 'youyou_4'); // 通过反射类ReflectionMethod调用指定实例的方法，并且传送参数
$value = $class->getMethod('getUsername')->invoke($instance);    // 通过反射类ReflectionMethod调用指定实例的方法
echo $value;echo "\n";   // 输出 youyou_4

try {
    $property = $class->getProperty('password_1');
    $property->setAccessible(true);   // 修改 $property 对象的可访问性
    $property->setValue($instance, 'password_2');  // 可以执行
    $value = $property->getValue($instance);     // 可以执行
    echo $value;echo "\n";   // 输出 password_2
    $class->getProperty('password')->setAccessible(true);    // 修改临时ReflectionProperty对象的可访问性
    $class->getProperty('password')->setValue($instance, 'password');// 不能执行，抛出不能访问异常
    $value = $class->getProperty('password')->getValue($instance);   // 不能执行，抛出不能访问异常
    $value = $instance->password;   // 不能执行，类本身的属性没有被修改，仍然是private
}catch(Exception $e){echo $e;}
```









