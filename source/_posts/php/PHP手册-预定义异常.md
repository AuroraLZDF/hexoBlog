---
title: PHP手册-预定义异常
date: 2018-10-31 11:10:41
tags: [PHP]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---
# Exception
Exception是所有异常的基类。

## 类摘要
```php
Exception {
/* 属性 */
protected string $message ;
protected int $code ;
protected string $file ;
protected int $line ;
/* 方法 */
public __construct ([ string $message = "" [, int $code = 0 [, Throwable $previous = NULL ]]] )
final public string getMessage ( void )
final public Throwable getPrevious ( void )
final public int getCode ( void )
final public string getFile ( void )
final public int getLine ( void )
final public array getTrace ( void )
final public string getTraceAsString ( void )
public string __toString ( void )
final private void __clone ( void )
}
```

## 属性
- message   异常消息内容
- code  异常代码
- file  抛出异常的文件名
- line  抛出异常在该文件中的行号

## 方法

### Exception::__construct
Exception::__construct — 异常构造函数

#### 说明

    public Exception::__construct ([ string $message = "" [, int $code = 0 [, Throwable $previous = NULL ]]] )
异常构造函数。

#### 参数
- message   抛出的异常消息内容。
- code   异常代码。
- previous  异常链中的前一个异常。

```bash
Note: 如果子类的 $code 和 $message 属性已设置，在调用 Exception 父类的构造器时可以省略默认参数。
```

### Exception::getMessage
Exception::getMessage — 获取异常消息内容
#### 说明
```php
final public string Exception::getMessage ( void )
```
返回异常消息内容。
#### 参数
此函数没有参数。

#### 返回值
返回字符串类型的异常消息内容。
#### 范例
```php
<?php
try {
    throw new Exception("Some error message");
} catch(Exception $e) {
    echo $e->getMessage();
}
```
以上例程的输出类似于：

    Some error message

### Exception::getPrevious
Exception::getPrevious — 返回异常链中的前一个异常
#### 说明
    final public Throwable Exception::getPrevious ( void )
返回异常链中的前一个异常（Exception::__construct()方法的第三个参数）。

#### 参数
此函数没有参数。

#### 返回值
返回异常链中的前一个异常 Throwable，否则返回NULL。
#### 范例
```php
<?php
class MyCustomException extends Exception {}

function doStuff() {
    try {
        throw new InvalidArgumentException("You are doing it wrong!", 112);
    } catch(Exception $e) {
        throw new MyCustomException("Something happend", 911, $e);
    }
}


try {
    doStuff();
} catch(Exception $e) {
    do {
        printf("%s:%d %s (%d) [%s]\n", $e->getFile(), $e->getLine(), $e->getMessage(), $e->getCode(), get_class($e));
    } while($e = $e->getPrevious());
}
```
以上例程的输出类似于：

    /home/bjori/ex.php:8 Something happend (911) [MyCustomException]
    /home/bjori/ex.php:6 You are doing it wrong! (112) [InvalidArgumentException]

### Exception::getCode
Exception::getCode — 获取异常代码
#### 说明
```php
final public int Exception::getCode ( void )
```
返回异常代码。
#### 参数
此函数没有参数。

#### 返回值
Exception 返回整型 `integer` 的异常代码，但在其他类中可能返回其他类型(比如在 PDOException 中返回 `string`)。
#### 范例
```php
<?php
try {
    throw new Exception("Some error message", 30);
} catch(Exception $e) {
    echo "The exception code is: " . $e->getCode();
}
```
以上例程的输出类似于：

    The exception code is: 30   

### Exception::getFile
Exception::getFile — 创建异常时的程序文件名称
#### 说明 
```php
final public string Exception::getFile ( void )
```
获取创建异常的程序文件名称。

#### 参数
此函数没有参数。

#### 返回值
返回发生异常的程序文件名称。

#### 范例
```php
<?php
try {
    throw new Exception;
} catch(Exception $e) {
    echo $e->getFile();
}
```
以上例程的输出类似于：

    /home/bjori/tmp/ex.php

### Exception::getLine
Exception::getLine — 获取创建的异常所在文件中的行号
#### 说明 
```php
final public int Exception::getLine ( void )
```
返回发生异常的代码在文件中的行号。    

#### 参数
此函数没有参数。

#### 返回值
返回发生异常的代码在文件中的行号。

#### 范例
```php
<?php
try {
    throw new Exception("Some error message");
} catch(Exception $e) {
    echo "The exception was thrown on line: " . $e->getLine();
}
```
以上例程的输出类似于：

    The exception was thrown on line: 3

### Exception::getTrace
Exception::getTrace — 获取异常追踪信息
#### 说明
```php
final public array Exception::getTrace ( void )
```
返回异常追踪信息。
#### 参数
此函数没有参数。

#### 返回值
返回包含异常追踪信息的数组（array）。

#### 范例
```php
<?php
function test() {
 throw new Exception;
}

try {
 test();
} catch(Exception $e) {
 var_dump($e->getTrace());
}
```
以上例程的输出类似于：

    array(1) {
    [0]=>
    array(4) {
        ["file"]=>
        string(22) "/home/bjori/tmp/ex.php"
        ["line"]=>
        int(7)
        ["function"]=>
        string(4) "test"
        ["args"]=>
        array(0) {
        }
    }
    }

### Exception::getTraceAsString
Exception::getTraceAsString — 获取字符串类型的异常追踪信息
#### 说明
```php
final public string Exception::getTraceAsString ( void )
```
以字符串类型返回异常追踪信息。

#### 参数
此函数没有参数。

#### 返回值
以字符串类型返回异常追踪信息。

#### 范例
```php
<?php
function test() {
    throw new Exception;
}

try {
    test();
} catch(Exception $e) {
    echo $e->getTraceAsString();
}
```
以上例程的输出类似于：

    #0 /home/bjori/tmp/ex.php(7): test()
    #1 {main}

### Exception::__toString
Exception::__toString — 将异常对象转换为字符串

#### 说明
```php
public string Exception::__toString ( void )
```
返回转换为字符串（string）类型的异常。

#### 参数
此函数没有参数。

#### 返回值
返回转换为字符串（string）类型的异常。

#### 范例
```php
<?php
try {
    throw new Exception("Some error message");
} catch(Exception $e) {
    echo $e;
}
```
以上例程的输出类似于：

    exception 'Exception' with message 'Some error message' in /home/bjori/tmp/ex.php:3
    Stack trace:
    #0 {main}

### Exception::__clone
Exception::__clone — 异常克隆

#### 说明
```php
final private void Exception::__clone ( void )
```
尝试克隆异常，这将导致一个致命错误。

#### 参数
此函数没有参数。

#### 返回值
没有返回值。

#### 错误／异常
异常被不允许克隆。





