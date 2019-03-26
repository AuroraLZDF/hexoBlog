---
title: Golang-sort排序
date: 2018-11-23 14:17:31
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---
sort包提供了排序切片和用户自定义数据集的函数。

# type Interface
```go
type Interface interface {
    // Len方法返回集合中的元素个数
    Len() int
    // Less方法报告索引i的元素是否比索引j的元素小
    Less(i, j int) bool
    // Swap方法交换索引i和j的两个元素
    Swap(i, j int)
}
```
一个满足 `sort.Interface` 接口的（集合）类型可以被本包的函数进行排序。方法要求集合中的元素可以被整数索引。

# type **IntSlice**
```go
type IntSlice []int
```
`IntSlice` 给 `[]int` 添加方法以满足 `Interface` 接口，以便排序为递增序列。

## func **(IntSlice) Len**
```go
func (p IntSlice) Len() int
```
`Len` 方法返回集合中的元素个数

## func **(IntSlice) Less**
```go
func (p IntSlice) Less(i, j int) bool
```
`Less` 方法报告索引 `i` 的元素是否比索引 `j` 的元素小

## func **(IntSlice) Swap**
```go
func (p IntSlice) Swap(i, j int)
```
`Swap` 方法交换索引 `i` 和 `j` 的两个元素

## func **(IntSlice) Sort**
```go
func (p IntSlice) Sort()
```
`Sort` 等价于调用 `Sort(p)`

## func **(IntSlice) Search**
```go
func (p IntSlice) Search(x int) int
```
`Search` 等价于调用 `SearchInts(p, x)`


# type **Float64Slice**
```go
type Float64Slice []float64
```
`Float64Slice` 给 `[]float64` 添加方法以满足 `Interface` 接口，以便排序为递增序列。

## func **(Float64Slice) Len**
```go
func (p Float64Slice) Len() int
```
## func **(Float64Slice) Less**
```go
func (p Float64Slice) Less(i, j int) bool
```
## func **(Float64Slice) Swap**
```go
func (p Float64Slice) Swap(i, j int)
```
## func **(Float64Slice) Sort**
```go
func (p Float64Slice) Sort()
```
`Sort` 等价于调用 `Sort(p)`

## func **(Float64Slice) Search**
```go
func (p Float64Slice) Search(x float64) int
```
`Search` 等价于调用 `SearchFloat64s(p, x)`


# type **StringSlice**
```go
type StringSlice []string
```
`StringSlice` 给 `[]string` 添加方法以满足 `Interface` 接口，以便排序为递增序列。

## func **(StringSlice) Len**
```go
func (p StringSlice) Len() int
```
## func **(StringSlice) Less**
```go
func (p StringSlice) Less(i, j int) bool
```
## func **(StringSlice) Swap**
```go
func (p StringSlice) Swap(i, j int)
```
## func **(StringSlice) Sort**
```go
func (p StringSlice) Sort()
```
`Sort` 等价于调用 `Sort(p)`

## func **(StringSlice) Search**
```go
func (p StringSlice) Search(x string) int
```
`Search` 等价于调用 `SearchStrings(p, x)`

# func **Sort**
```go
func Sort(data Interface)
```
`Sort` 排序 `data`。它调用1次 `data.Len` 确定长度，调用 `O(n*log(n))` 次 `data.Less` 和 `data.Swap`。本函数不能保证排序的稳定性（即不保证相等元素的相对次序不变）。

# func **Stable**
```go
func Stable(data Interface)
```
`Stable` 排序 `data`，并保证排序的稳定性，相等元素的相对次序不变。

它调用1次 `data.Len`，`O(n*log(n))` 次 `data.Less` 和 `O(n*log(n)*log(n))` 次 `data.Swap`。

# func **IsSorted**
```go
func IsSorted(data Interface) bool
```
`IsSorted` 报告 `data` 是否已经被排序。

# func **Reverse**
```go
func Reverse(data Interface) Interface
```
`Reverse` 包装一个 `Interface `接口并返回一个新的 `Interface` 接口，对该接口排序可生成递减序列。

示例：
```go
package main

import (
	"sort"
	"fmt"
)

func main() {
	s := []int{5, 2, 6, 3, 1, 4} // unsorted
	sort.Sort(sort.Reverse(sort.IntSlice(s)))
	fmt.Println(s)
}
```
输出：

    [6 5 4 3 2 1]

# func **Search**    
```go
func Search(n int, f func(int) bool) int
```
`Search` 函数采用*二分法*搜索找到[0, n)区间内最小的满足f(i)==true的值i。也就是说，`Search` 函数希望 `f` 在输入位于区间[0, n)的前面某部分（可以为空）时返回假，而在输入位于剩余至结尾的部分（可以为空）时返回真；`Search` 函数会返回满足f(i)==true的最小值i。如果没有该值，函数会返回n。注意，未找到时的返回值不是-1，这一点和 `strings.Index` 等函数不同。`Search` 函数只会用区间[0, n)内的值调用 `f`。

一般使用 `Search` 找到值x在插入一个有序的、可索引的数据结构时，应插入的位置。这种情况下，参数 `f`（通常是闭包）会捕捉应搜索的值和被查询的数据集。

例如，给定一个递增顺序的切片，调用 `Search(len(data), func(i int) bool { return data[i] >= 23 })` 会返回data中最小的索引i满足data[i] >= 23。如果调用者想要知道23是否在切片里，它必须另外检查data[i] == 23。

搜索递减顺序的数据时，应使用<=运算符代替>=运算符。

下列代码尝试在一个递增顺序的整数切片中找到值x：
```go
func GuessingGame() {
	var s string
	fmt.Printf("Pick an integer from 0 to 100.\n")
	answer := sort.Search(100, func(i int) bool {
		fmt.Printf("Is your number <= %d? ", i)
		fmt.Scanf("%s", &s)
		return s != "" && s[0] == 'y'
	})
	fmt.Printf("Your number is %d.\n", answer)
}
```
# Ints

## func **Ints**
```go
func Ints(a []int)
```
`Ints` 函数将 `a` 排序为递增顺序。

示例：
```go
package main

import (
	"sort"
	"fmt"
)

func main() {
	s := []int{5, 2, 6, 3, 1, 4} // unsorted
	sort.Ints(s)
	fmt.Println(s)
}
```
输出：

    [1 2 3 4 5 6]

## func **IntsAreSorted**
```go
func IntsAreSorted(a []int) bool
```
`IntsAreSorted` 检查 `a` 是否已排序为递增顺序。    
示例：
```go
package main

import (
	"sort"
	"fmt"
)

func main() {
	s := []int{5, 2, 6, 3, 1, 4} // unsorted
	sort.Ints(s)
    fmt.Println(s)
    fmt.Println(sort.IntsAreSorted(s))
}
```
输出：

    [1 2 3 4 5 6]
    true

## func **SearchInts**
```go
func SearchInts(a []int, x int) int
```
`SearchInts` 在递增顺序的 `a` 中搜索 `x`，返回 `x` 的索引。如果查找不到，返回值是 `x`应该插入 `a` 的位置（以保证a的递增顺序），返回值可以是len(a)。

# Float64s

## func **Float64s**
```go
func Float64s(a []float64)
```
`Float64s` 函数将 a` 排序为递增顺序。

## func **Float64sAreSorted**
```go
func Float64sAreSorted(a []float64) bool
```
`Float64sAreSorted` 检查 `a`是否已排序为递增顺序。

## func **SearchFloat64s**
```go
func SearchFloat64s(a []float64, x float64) int
```
`SearchFloat64s` 在递增顺序的 `a` 中搜索 `x`，返回 `x` 的索引。如果查找不到，返回值是 `x` 应该插入 `a` 的位置（以保证a的递增顺序），返回值可以是len(a)。


# Strings

## func **Strings**
```go
func Strings(a []string)
```
`Strings` 函数将 `a` 排序为递增顺序。

## func **StringsAreSorted**
```go
func StringsAreSorted(a []string) bool
```
`StringsAreSorted` 检查 `a` 是否已排序为递增顺序。

## func **SearchStrings**
```go
func SearchStrings(a []string, x string) int
```
`SearchStrings` 在递增顺序的 `a` 中搜索 `x`，返回 `x` 的索引。如果查找不到，返回值是 `x` 应该插入 `a` 的位置（以保证a的递增顺序），返回值可以是len(a)。
