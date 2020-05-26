---
title: PHP手册-预定义接口
date: 2018-10-31 14:17:59
tags: [PHP]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---
# Traversable（遍历）接口
## 简介
检测一个类是否可以使用 foreach 进行遍历的接口。

无法被单独实现的基本抽象接口。相反它必须由 `IteratorAggregate` 或 `Iterator` 接口实现。

    Note:
    实现此接口的内建类可以使用 foreach 进行遍历而无需实现 IteratorAggregate 或 Iterator 接口。
---
    Note:
    这是一个无法在 PHP 脚本中实现的内部引擎接口。IteratorAggregate 或 Iterator 接口可以用来代替它。

## 接口摘要
```php
Traversable {
}
```
这个接口没有任何方法，它的作用仅仅是作为所有可遍历类的基本接口。

## 范例
```php
<?php
$myarray = array('one', 'two', 'three');
$myobj = (object)$myarray;

if ( !($myarray instanceof \Traversable) ) {
    print "myarray is NOT Traversable";
}
if ( !($myobj instanceof \Traversable) ) {
    print "myobj is NOT Traversable";
}

foreach ($myarray as $value) {
    print $value;
}
foreach ($myobj as $value) {
    print $value;
}
```
上面的例子大概会输出：

    myarray is NOT Traversable
    myobj is NOT Traversable
    one
    two
    three
    one
    two
    three

# Iterator（迭代器）接口
##　简介
可在内部迭代自己的外部迭代器或类的接口。

## 接口摘要 
```php
Iterator extends Traversable {
/* 方法 */
abstract public mixed current ( void )
abstract public scalar key ( void )
abstract public void next ( void )
abstract public void rewind ( void )
abstract public bool valid ( void )
}
```

## 预定义迭代器
PHP 已经提供了一些用于日常任务的迭代器。 详细列表参见 [SPL 迭代器](http://php.net/manual/zh/spl.iterators.php)。

## 范例
这个例子展示了使用 foreach 时，迭代器方法的调用顺序。
```php
<?php
class myIterator implements Iterator {
    private $position = 0;
    private $array = array(
        "firstelement",
        "secondelement",
        "lastelement",
    );  

    public function __construct() {
        $this->position = 0;
    }

    function rewind() {
        var_dump(__METHOD__);
        $this->position = 0;
    }

    function current() {
        var_dump(__METHOD__);
        return $this->array[$this->position];
    }

    function key() {
        var_dump(__METHOD__);
        return $this->position;
    }

    function next() {
        var_dump(__METHOD__);
        ++$this->position;
    }

    function valid() {
        var_dump(__METHOD__);
        return isset($this->array[$this->position]);
    }
}

$it = new myIterator;

foreach($it as $key => $value) {
    var_dump($key, $value);
    echo "\n";
}
```
以上例程的输出类似于：
```bash
string(18) "myIterator::rewind"
string(17) "myIterator::valid"
string(19) "myIterator::current"
string(15) "myIterator::key"
int(0)
string(12) "firstelement"

string(16) "myIterator::next"
string(17) "myIterator::valid"
string(19) "myIterator::current"
string(15) "myIterator::key"
int(1)
string(13) "secondelement"

string(16) "myIterator::next"
string(17) "myIterator::valid"
string(19) "myIterator::current"
string(15) "myIterator::key"
int(2)
string(11) "lastelement"

string(16) "myIterator::next"
string(17) "myIterator::valid"
```
## Table of Contents
- Iterator::current — 返回当前元素
- Iterator::key — 返回当前元素的键
- Iterator::next — 向前移动到下一个元素
- Iterator::rewind — 返回到迭代器的第一个元素
- Iterator::valid — 检查当前位置是否有效

*[php 迭代接口的作用](https://segmentfault.com/q/1010000010830185)*

# IteratorAggregate（聚合式迭代器）接口
## 简介 
创建外部迭代器的接口。
## 接口摘要 
```php
IteratorAggregate extends Traversable {
/* 方法 */
abstract public Traversable getIterator ( void )    // 获取一个外部迭代器
}
```
### Example #1 基本用法
```php
<?php
class myData implements IteratorAggregate {
    public $property1 = "Public property one";
    public $property2 = "Public property two";
    public $property3 = "Public property three";

    public function __construct() {
        $this->property4 = "last property";
    }

    public function getIterator() {
        return new ArrayIterator($this);
    }
}

$obj = new myData;

foreach($obj as $key => $value) {
    var_dump($key, $value);
    echo "\n";
}
```
以上例程的输出类似于：

    string(9) "property1"
    string(19) "Public property one"
    
    string(9) "property2"
    string(19) "Public property two"
    
    string(9) "property3"
    string(21) "Public property three"
    
    string(9) "property4"
    string(13) "last property"

# ArrayAccess（数组式访问）接口
## 简介
提供像**访问数组一样访问对象**的能力的接口。
## 接口摘要
```php
ArrayAccess {
/* 方法 */
abstract public boolean offsetExists ( mixed $offset )
abstract public mixed offsetGet ( mixed $offset )
abstract public void offsetSet ( mixed $offset , mixed $value )
abstract public void offsetUnset ( mixed $offset )
}
```
## Example #1 Basic usage
```php
<?php
class obj implements arrayaccess {
    private $container = array();
    public function __construct() {
        $this->container = array(
            "one"   => 1,
            "two"   => 2,
            "three" => 3,
        );
    }
    public function offsetSet($offset, $value) {
        if (is_null($offset)) {
            $this->container[] = $value;
        } else {
            $this->container[$offset] = $value;
        }
    }
    public function offsetExists($offset) {
        return isset($this->container[$offset]);
    }
    public function offsetUnset($offset) {
        unset($this->container[$offset]);
    }
    public function offsetGet($offset) {
        return isset($this->container[$offset]) ? $this->container[$offset] : null;
    }
}

$obj = new obj;

var_dump(isset($obj["two"]));
var_dump($obj["two"]);
unset($obj["two"]);
var_dump(isset($obj["two"]));
$obj["two"] = "A value";
var_dump($obj["two"]);
$obj[] = 'Append 1';
$obj[] = 'Append 2';
$obj[] = 'Append 3';
print_r($obj);
```
以上例程的输出类似于：

    bool(true)
    int(2)
    bool(false)
    string(7) "A value"
    obj Object
    (
        [container:obj:private] => Array
            (
                [one] => 1
                [three] => 3
                [two] => A value
                [0] => Append 1
                [1] => Append 2
                [2] => Append 3
            )
    
    )

## Table of Contents ¶
- ArrayAccess::offsetExists — 检查一个偏移位置是否存在
- ArrayAccess::offsetGet — 获取一个偏移位置的值
- ArrayAccess::offsetSet — 设置一个偏移位置的值
- ArrayAccess::offsetUnset — 复位一个偏移位置的值

# 序列化接口
## 简介
自定义序列化的接口。

实现此接口的类将不再支持 __sleep() 和 __wakeup()。不论何时，只要有实例需要被序列化，serialize 方法都将被调用。它将不会调用 __destruct() 或有其他影响，除非程序化地调用此方法。当数据被反序列化时，类将被感知并且调用合适的 unserialize() 方法而不是调用 __construct()。如果需要执行标准的构造器，你应该在这个方法中进行处理。

## 接口摘要
```php
Serializable {
/* 方法 */
abstract public string serialize ( void )
abstract public mixed unserialize ( string $serialized )
}
```
## Example #1 Basic usage
```php
<?php
class obj implements Serializable {
    private $data;
    public function __construct() {
        $this->data = "My private data";
    }
    public function serialize() {
        return serialize($this->data);
    }
    public function unserialize($data) {
        $this->data = unserialize($data);
    }
    public function getData() {
        return $this->data;
    }
}

$obj = new obj;
$ser = serialize($obj);

$newobj = unserialize($ser);

var_dump($newobj->getData());
```
以上例程的输出类似于：

    string(15) "My private data"
## Table of Contents
- Serializable::serialize — 对象的字符串表示
- Serializable::unserialize — 构造对象

# Closure 类
## 简介
用于代表 匿名函数 的类.

匿名函数（在 PHP 5.3 中被引入）会产生这个类型的对象。在过去，这个类被认为是一个实现细节，但现在可以依赖它做一些事情。自 PHP 5.4 起，这个类带有一些方法，允许在匿名函数创建后对其进行更多的控制。

除了此处列出的方法，还有一个 __invoke 方法。这是为了与其他实现了 __invoke()魔术方法 的对象保持一致性，但调用匿名函数的过程与它无关。

## 类摘要
```php
Closure {
    /* 方法 */
    __construct ( void )
    public static Closure bind ( Closure $closure , object $newthis [, mixed $newscope = 'static' ] )
    public Closure bindTo ( object $newthis [, mixed $newscope = 'static' ] )
}
```
## Table of Contents
- Closure::__construct — 用于禁止实例化的构造函数
- Closure::bind — 复制一个闭包，绑定指定的$this对象和类作用域。
- Closure::bindTo — 复制当前闭包对象，绑定指定的$this对象和类作用域。

# 生成器类
## 简介
Generator 对象是从 generators返回的.

    Caution Generator 对象不能通过 new 实例化.
## 类摘要
```php
Generator implements Iterator {
/* 方法 */
public mixed current ( void )
public mixed key ( void )
public void next ( void )
public void rewind ( void )
public mixed send ( mixed $value )
public void throw ( Exception $exception )
public bool valid ( void )
public void __wakeup ( void )
}
```
## Table of Contents
- Generator::current — 返回当前产生的值
- Generator::key — 返回当前产生的键
- Generator::next — 生成器继续执行
- Generator::rewind — 重置迭代器
- Generator::send — 向生成器中传入一个值
- Generator::throw — 向生成器中抛入一个异常
- Generator::valid — 检查迭代器是否被关闭
- Generator::__wakeup — 序列化回调



