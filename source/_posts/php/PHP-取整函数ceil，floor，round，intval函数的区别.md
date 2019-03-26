---
title: PHP-取整函数ceil，floor，round，intval函数的区别
date: 2019-02-21 14:44:29
tags: [PHP]
categories: [PHP、笔记]
toc: true
cover: '/images/categories/php.jpeg'
---

# `ceil` — 进一法取整
**说明**

```php
float ceil ( float $value )
```
返回不小于 `value` 的下一个整数，`value` 如果有小数部分则进一位。`ceil()` 返回的类型仍然是 `float`，因为 `float` 值的范围通常比 `integer` 要大。

**示例**：

```php
<?php
    echo ceil(4.3); // 5

    echo ceil(9.999); // 10
?>
```

# `floor` — 舍去法取整
**说明**

```php
float floor ( float $value )
```
返回不大于 `value` 的下一个整数，将 `value` 的小数部分舍去取整。`floor()` 返回的类型仍然是 `float`，因为 `float` 值的范围通常比 `integer` 要大。

**示例**：

```php
<?php
    echo floor(4.3); // 4

    echo floor(9.999); // 9
?>
```

# `round` — 对浮点数进行四舍五入
**说明**

```php
float round ( float $val [, int $precision ] )
```
返回将 `val` 根据指定精度 `precision`（十进制小数点后数字的数目）进行四舍五入的结果。`precision` 也可以是负数或零（默认值）。

**示例**：

```php
<?php
    echo round(3.4); // 3

    echo round(3.5); // 4

    echo round(3.6); // 4

    echo round(3.6, 0); // 4

    echo round(1.95583, 2); // 1.96

    echo round(1241757, -3); // 1242000

    echo round(5.045, 2); // 5.05

    echo round(5.055, 2); // 5.06

?>
```
**注意**： PHP 默认不能正确处理类似 “12,300.2″ 的字符串。

**注意**： precision 参数是在 PHP 4 中被引入的。

# `intval` — 获取变量的整数值
**说明**

```php
int intval ( mixed $var [, int $base ] )
```
通过使用特定的进制转换（默认是十进制），返回变量 `var` 的 `integer` 数值。
`var` 可以是任何标量类型。`intval()` 不能用于 `array` 或 `object`。

**示例**：

```php
<?php
    echo intval(4.3);     // 4

    echo intval(9.999); // 9
?>
 ```

**注意**：
除非 `var` 参数是字符串，否则 `intval()` 的 `base` 参数不会有效果。

`floor` 函数与 `intval` 函数功能相同，所不同之处是一个返回的浮点数(float)，而另一个是整数(integer)，因为 `float` 值的范围通常比 `integer` 要大。不过就数值本身来说两者是相等的。






