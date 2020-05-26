---
title: PHP5垃圾回收机制
date: 2020-05-26 14:29:25
categories: [PHP]
tags: [PHP]
---

### 概念

**垃圾回收机制 是一种内存动态分配的方案，它会自动释放程序不再使用的已分配的内存块。**

**垃圾回收机制** 可以让程序员不必过分关心程序内存分配，从而将更多的精力投入到业务逻辑。

> 与之相关的一个概念，**内存泄露** 指的是程序未能释放那些已经不再使用的内存，造成内存的浪费。

那么 PHP 是如何实现垃圾回收机制的呢？

### PHP变量的内部存储结构

首先还是需要了解下 [基础知识](https://www.cnblogs.com/martini-d/p/php5variable.html)，便于对垃圾回收原理内容的理解。

*****PHP **所有类型**的变量在底层都会以 **zval 结构体** 的形式实现 (源码文件Zend/zend.h)*****

**PHP5 中 zval 结构体定义如下**：

```c
struct _zval_struct {
    /* Variable information */
    zvalue_value value;     /* 变量value值 */
    zend_uint refcount__gc; /* 引用计数内存中使用次数，为0删除该变量 */
    zend_uchar type;    /* 变量类型 */
    zend_uchar is_ref__gc; /* 区分是否是引用变量，是引用为1，否则为0 */
};
```

注：上面 zval 结构体是 php5.3 版本之后的结构，php5.3 之前因为没有引入新的垃圾回收机制，即 **GC**，所以命名也没有`_gc`；而 php7 版本之后由于性能问题所以改写了 zval 结构。

### 引用计数原理

每个 php 变量存在一个叫 "zval" 的变量容器中。一个 **zval** 变量容器，除了包含变量的类型和值，还包括两个字节的额外信息。第一个是 "is_ref"，是个 `bool` 值，用来标识这个变量是否是属于引用集合(reference set)。通过这个字节，php 引擎才能把普通变量和引用变量区分开来。由于 php 允许用户通过使用 `&` 来使用自定义引用**zval** 变量容器中还有一个内部引用计数机制，来优化内存使用。第二个额外字节是 "refcount"，用以表示指向这个 **zval** 变量容器的变量(也称符号即 symbol)个数。所有的符号存在一个符号表中，其中每个符号都有作用域(scope)，那些主脚本(比如：通过浏览器请求的的脚本)和每个函数或者方法也都有作用域。

当一个变量被赋常量值时，就会生成一个zval变量容器，如下例这样

```php
<?php $a = "new string";
```

在上例中，新的变量 *a*，是在当前作用域中生成的。并且生成了类型为 [string](https://www.php.net/manual/zh/language.types.string.php) 和值为 *new string* 的变量容器。在额外的两个字节信息中，"is_ref" 被默认设置为 **`FALSE`**，因为没有任何自定义的引用生成。"refcount" 被设定为 *1*，因为这里只有一个变量使用这个变量容器. 注意到当 "refcount" 的值是 *1* 时，"is_ref" 的值总是**`FALSE`**. 如果你已经安装了[» Xdebug](http://xdebug.org/)，你能通过调用函数 **xdebug_debug_zval()**显示"refcount"和"is_ref"的值。

```php
<?php xdebug_debug_zval('a');
```

以上例程会输出：

```bash
# php-5.3
a: (refcount=1, is_ref=0)='new string'
# php-7.3
a: (interned, is_ref=0)='new string'
```

```php
<?php
$a = "new string";
$b = $a;
xdebug_debug_zval( 'a', 'b' );
```

以上例程会输出：

```bash
# php-5.3
a: (refcount=2, is_ref=0)='new string'
b: (refcount=1, is_ref=0)='new string'
# php-7.3
a: (interned, is_ref=0)='new string'
b: (interned, is_ref=0)='new string'
```

这时，引用次数是*2*，因为同一个变量容器被变量 a 和变量 b关联.当没必要时，php不会去复制已生成的变量容器。变量容器在”refcount“变成0时就被销毁。当任何关联到某个变量容器的变量离开它的作用域(比如：函数执行结束)，或者对变量调用了函数 [unset()](https://www.php.net/manual/zh/function.unset.php)时，”refcount“就会减1。



在 PHP7 中，**zval** 结构体中有一个标志来决定 **zval** 是否能被引用计数。
像 `null`,`bool`,`int`,`double` 这些变量类型永远不会被引用计数（这个地方可能有些不太严谨，鸟哥的博客中写道 PHP7 中 **zval** 的类型共有 18 种，其中 **IS_LONG**,**IS_DOUBLE**,**IS_NULL**,**IS_FALSE**,**IS_TRUE** 不会使用引用计数）。像 `object`,`resources`,`references` 这些变量类型总是会使用引用计数。
然而，像 `array`，`strings` 这些变量类型有时会使用引用计数，有时则不会。

不使用引用计数的字符串类型被叫做 “interned string（保留字符串）”。如果你使用一个NTS(非线程安全)的PHP7来构建，通常情况下，代码中的所有字符串文字都将是限定的。这些保留字符串都是不可重复的（即，只会存在一个含有特定内容的保留字符串）。它会一直存在直到请求结束时才销毁，所以也就无需进行引用计数。如果使用了 opcache 的话，保留字符会被存储在共享内存中，在这种情况下，无法使用引用计数（因为我们引用计数的机制是非原子的）。保留字符串的伪引用计数为 1。

对于数组来说，无引用计数的变量称为“不可变数组”。如果使用 opcache，则代码中的常量数组文字将转换为不可变数组。同样的，他们存在于共享内存中，因此不得使用引用计数。不可变数组的伪引用数为2，因为它允许我们优化某些分离路径。



**PHP5 中 `value` 的结构定义如下**

```c
typedef union _zvalue_value {
    long lval;                 // 用于 bool 类型、整型和资源类型
    double dval;               // 用于浮点类型
    struct {                   // 用于字符串
        char *val;			   // 字符内容
        int len;			   // 字符长度
    } str;
    HashTable *ht;             // 用于数组
    zend_object_value obj;     // 用于对象
    zend_ast *ast;             // 用于常量表达式(PHP5.6 才有)
} zvalue_value;
```

**PHP7 中的 zval**

```c
struct _zval_struct {
	zend_value        value;			/* value */
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar    type,			/* active type */
				zend_uchar    type_flags,
				zend_uchar    const_flags,
				zend_uchar    reserved)	    /* call info for EX(This) */
		} v;
		uint32_t type_info;
	} u1;
	union {
		uint32_t     var_flags;
		uint32_t     next;                 /* hash collision chain */
		uint32_t     cache_slot;           /* literal cache slot */
		uint32_t     lineno;               /* line number (for ast nodes) */
		uint32_t     num_args;             /* arguments number for EX(This) */
		uint32_t     fe_pos;               /* foreach position */
		uint32_t     fe_iter_idx;          /* foreach iterator index */
	} u2;
};
```

**PHP7 中 `value` 的结构定义**

```c
typedef union _zend_value {
	zend_long         lval;				/* long value */
	double            dval;				/* double value */
	zend_refcounted  *counted;
	zend_string      *str;
	zend_array       *arr;
	zend_object      *obj;
	zend_resource    *res;
	zend_reference   *ref;
	zend_ast_ref     *ast;
	zval             *zv;
	void             *ptr;
	zend_class_entry *ce;
	zend_function    *func;
	struct {
		uint32_t w1;
		uint32_t w2;
	} ww;
} zend_value;
```

首先需要注意的是现在 value 联合体需要的内存是 8 个字节而不是 16。它只会直接存储整型（`lval`）或者浮点型（`dval`）数据，其他情况下都是指针（上面提到过，指针占用 8 个字节，最下面的结构体由两个 4 字节的无符号整型组成）。上面所有的指针类型（除了特殊标记的）都有一个同样的头（`zend_refcounted`）用来存储引用计数：

```c
typedef struct _zend_refcounted_h {
	uint32_t         refcount;			/* reference counter 32-bit */
	union {
		struct {
			ZEND_ENDIAN_LOHI_3(
				zend_uchar    type,
				zend_uchar    flags,    /* used for strings & objects */
				uint16_t      gc_info)  /* keeps GC root number (or 0) and color */
		} v;
		uint32_t type_info;
	} u;
} zend_refcounted_h;
```

[引用计数基本知识](https://www.php.net/manual/zh/features.gc.refcounting-basics.php)

[回收周期](https://www.php.net/manual/zh/features.gc.collecting-cycles.php)

[PHP5底层原理之变量](https://www.cnblogs.com/martini-d/p/php5variable.html)

