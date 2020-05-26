---
title: PHP-子类重写父类属性问题
date: 2019-04-15 17:26:07
tags: [Linux,PHP]
categories: [笔记]
toc: true
cover: '/images/categories/php.jpeg'
---

### 问题还原

**新建父类文件**

```php
<?php
class Animal
{
	public $hasLeg = false;
    public function __consruct()
    {
    }
}
```

**新建子类文件**

```php
<?php
class Dog
{
    protected $hasLeg = true;
}
```

**执行脚本**

```bash
$ php Dog.php
PHP Fatal error:  Access level to Dog::$hasLeg must be public (as in class Animal) in /www/code/html/test/Dog.php on line 4

Fatal error: Access level to Dog::$hasLeg must be public (as in class Animal) in /www/code/html/test/Dog.php on line 4
```

经过比较父类、子类发现，子类在重在父类的 `$hasLeg` 属性时，将这个属性的访问权限由 `public`→`protected` 。尝试修改子类：

```php
<?php
class Dog
{
    // protected $hasLeg = true;
    public $hasLeg = true;
    public function __consruct()
    {
    }
}
```

**再次执行脚本**

```bash
$ php Dog.php

```

### 总结

PHP 子类在重在父类属性时，重载的属性的访问权限要大于等于父类属性的访问权限（`private` < `protected` < `public`），否则会报错！