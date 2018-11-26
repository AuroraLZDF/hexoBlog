---
title: 【转】Golang-通过内存缓存来提升性能
date: 2018-11-21 15:45:11
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

当在进行大量的计算时，提升性能最直接有效的一种方式就是**避免重复计算**。通过在内存中缓存和重复利用相同计算的结果，称之为内存缓存。最明显的例子就是生成斐波那契数列的程序。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	result := 0
	start := time.Now()

	for i := 0; i < 41; i++ {
		result = fibonacci(i)
		fmt.Printf("fibonacci(%d) is: %d\n", i, result)
	}

	end := time.Now()
	delta := end.Sub(start)
	fmt.Printf("longCalculation took this amount of time: %s\n", delta)
}

func fibonacci(n int) (res int) {
	if n <= 1 {
		res = 1
	} else {
		res = fibonacci(n-1) + fibonacci(n-2)
	}
	return
}
```
程序耗时：

    ...
    longCalculation took this amount of time: 2.750569874s

要计算数列中第 `n` 个数字，需要先得到之前两个数的值，但很明显绝大多数情况下前两个数的值都是已经计算过的。即每个更后面的数都是基于之前计算结果的重复计算。

而我们要做就是将第 `n` 个数的值存在数组中索引为 `n` 的位置（详见第 7 章），然后在数组中查找是否已经计算过，如果没有找到，则再进行计算。

示例：
```go
package main

import (
	"time"
	"fmt"
)

const LIM = 41

var fibs [LIM]uint64

func main() {
	var result uint64 = 0
	start := time.Now()

	for i := 0; i < LIM; i++ {
		result = fibonacci(i)
		fmt.Printf("fibonacci(%d) is: %d\n", i, result)
	}

	end := time.Now()
	delta := end.Sub(start)
	fmt.Printf("longCalculation took this amount of time: %s\n", delta)
}

func fibonacci(n int) (res uint64) {
	// memoization: check if fibonacci(n) is already known in array:
	if fibs[n] != 0 {
		res = fibs[n]
		return
	}

	if n <= 1 {
		res = 1
	} else {
		res = fibonacci(n-1) + fibonacci(n-2)
	}
	fibs[n] = res

	return
}
```
程序耗时：

    ...
    longCalculation took this amount of time: 69.255µs

下面是计算到第 40 位数字的性能对比：

- 普通写法：2.750569874 秒
- 内存缓存：0.000069255 秒

内存缓存的优势显而易见，而且您还可以将它应用到其它类型的计算中，例如使用 `map` 而不是数组或切片。

内存缓存的技术在使用计算成本相对昂贵的函数时非常有用（不仅限于例子中的递归），譬如**大量进行相同参数的运算**。这种技术还可以应用于纯函数中，即相同输入必定获得相同输出的函数。

【转】[通过内存缓存来提升性能](https://go.fdos.me/06.12.html)



