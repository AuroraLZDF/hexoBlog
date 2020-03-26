---
title: Golang-环境搭建
date: 2020-03-26 15:08:24
categories: [Golang]
tags: [Golang]
---

# 安装包下载
[go1.14.1.linux-amd64.tar.gz](https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz)

# 解压到指定目录下
```bash
$ tar -C /usr/local -xzf go1.14.1.linux-amd64.tar.gz
```

# 环境变量配置
环境变量设置可以选择 `/etc/profile` 或 `~/.bashrc`
```bash
$ dedit ~/.bashrc
```
Go环境变量配置如下
```bash
# golang
export GOROOT=/usr/local/go             # GOROOT: GO的安装路径
export PATH=$PATH:$GOROOT/bin           
export GOPATH=~/go                      # GOPATH: GO的工作路径
export GO111MODULE=on                   # 开始 Go Modules
```

# 刷新环境变量
```bash
$ source ~/.bashrc
```

# 测试
```bash
$ go env
GO111MODULE="on"
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/xxxx/.cache/go-build"
GOENV="/home/xxxx/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/home/xxxx/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD="/dev/null"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build248485667=/tmp/go-build -gno-record-gcc-switches"
```

# Go 工作空间
```bash
$ tree
.
├── bin             # 存放编译后生成的可执行文件
├── pkg             # 存放编译后生成的文件
└── src             # 存放源代码
```

# Hello World
代码
```bash
package main

import "fmt"
 
func main(){
    fmt.Printf("hello world\n")
}
```
执行
```bash
$ go run hello.go
hello world
```

至此，`Go` 开发环境搭建完成








