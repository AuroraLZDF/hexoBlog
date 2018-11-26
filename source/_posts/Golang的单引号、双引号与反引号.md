---
title: 【转】Golang的单引号、双引号与反引号
date: 2018-11-12 16:10:36
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

Go语言的字符串类型 `string` 在本质上就与其他语言的字符串类型不同：

- Java的String、C++的std::string以及Python3的str类型都只是 `定宽字符序列`

- Go语言的字符串是一个用 `UTF-8编码的变宽字符序列`，*它的每一个字符都用一个或多个字节表示*

即：**一个Go语言字符串是一个`任意字节`的常量序列**。

Golang的双引号和反引号都可用于表示一个常量字符串，不同在于：

- 双引号用来创建**可解析的字符串字面量**(支持转义，但不能用来引用多行)

- 反引号用来创建**原生的字符串字面量**，这些字符串可能由多行组成(不支持任何转义序列)，原生的字符串字面量多用于书写多行消息、HTML以及正则表达式。

而单引号则用于表示Golang的一个特殊类型：`rune`，类似其他语言的 `byte` 但又不完全一样，是指：**码点字面量（Unicode code point）**，不做任何转义的原始内容。


注：
本文转载自 — [Golang的单引号、双引号与反引号](https://studygolang.com/articles/8431)