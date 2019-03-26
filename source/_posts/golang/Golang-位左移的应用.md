---
title: 『记』Golang-位左移的应用
date: 2018-11-09 15:31:25
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

## 位左移 <<：
- 用法：`bitP << n`。
- bitP 的位向左移动 `n` 位，右侧空白部分使用 0 填充；如果 n 等于 2，则结果是 2 的相应倍数，即 2 的 n 次方。例如：
```go
  1 << 10 // 等于 1 KB
  1 << 20 // 等于 1 MB
  1 << 30 // 等于 1 GB
```

### 位左移常见实现存储单位的用例
使用位左移与 iota 计数配合可优雅地实现存储单位的常量枚举：
```go
type ByteSize float64
const (
    _ = iota // 通过赋值给空白标识符来忽略值
    KB ByteSize = 1<<(10*iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

