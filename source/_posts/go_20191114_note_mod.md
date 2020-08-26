---
layout: post
title: Go Module
date: 2019-11-14 09:29
catalog: true
categories: GoLang
tags: 笔记

---




上一篇记录了Go语言的基础知识。这里对Go的模块管理进行介绍。



### 使用Module

#### Module 初始化

使用`go mod init xx ` 来初始化创建`go.mod`文件。

注意**在`$GOPATH/src`之外的目录调用go mod init命令，且未设置环境变量GO111MODULE时，会出现错误**，需要手动激活`export GO111MODULE=on ` 。这里我们创建应用`module-test`，使用该应用进行测试。命令如下：

```bash
liangtongdeiMac:~ liangtong$ cd /Users/liangtong/Documents/go/src/module-test 
liangtongdeiMac:module-test liangtong$ go mod init modtest
go: modules disabled inside GOPATH/src by GO111MODULE=auto; see 'go help modules'
liangtongdeiMac:module-test liangtong$ export GO111MODULE=on 
liangtongdeiMac:module-test liangtong$ go mod init modtest
go: creating new go.mod: module modtest
```

此时当前应用目录下，会出现一个`go.mod`文件，此时文件包含一行内容

```mod
module modtest
```



#### 添加依赖

新建`hello.go`文件，添加代码，这里我们代码中依赖一个github上的模块，如下

```go
package main

import(
    "fmt"
    goTest "github.com/liangtongdev/go-test"
)

func Hello () string{
    s := goTest.Hello()
    fmt.Println(s)
    return s
}
```

添加测试文件`hello_test.go`，代码如下

```go
package main
import "testing"

func TestHello(t *testing.T) {
    want := "Hello, world."
    if got := Hello(); got != want {
        t.Errorf("Hello() = %q, want %q", got, want)
    }
}
```

在应用`module-test`根目录，执行 `go test`命令，命令行如下

```bash
liangtongdeiMac:module-test liangtong$ go test
Hello, world.
PASS
ok  	modtest	0.503s
liangtongdeiMac:module-test liangtong$ 
```

此时 `go.mod`文件的内容如下

```go.mod
module modtest

require github.com/liangtongdev/go-test v1.2.0
```

另外，目录中多了一个`go.sum`文件，内容记录的是一些主要的版本信息，内容如下

```go.sum
github.com/liangtongdev/go-test v1.2.0 h1:+nsknGYi0fRJ2UPCoHrp8ehNM7Prf0/bCFeUkTHNXYg=
github.com/liangtongdev/go-test v1.2.0/go.mod h1:DA9vwQSXNcnkl8UDsUDRI1tiVQUlIf/bphhItSDC7WM=
```



#### 更新依赖

在`go.mod`文件中，会记录模块的版本信息，例如`require github.com/liangtongdev/go-test v1.2.0` 中的 `v1.2.0`版本。默认情况下，获取的是最新的版本

```bash
go get github.com/liangtongdev/go-test
```

当然如果需要特定的版本，可以在命令中指定版本信息

```bahs
go get github.com/liangtongdev/go-test@v1.2.0
```



#### 移除无用的依赖

可以通过命令`go list -m all` 查看`go.mod`中所有的依赖模块

```bash
go list -m all
```

通过`go mod tidy`命令，移除无用的依赖包

```bash
go mod tidy
```



### 创建Module

这里我们借助于`github`来创建自己的module，首先在github上创建一个代码仓库。编写自己的代码，测试。

一切正常的时候，我们创建tag并发布

```bash
go mod init github.com/liangtongdev/go-test

git add *
git commit -m "module init"
git push origin master
git tag v1.2.0
git push origin v1.2.0
```

其中版本信息格式为`vMAJOR.MINOR.PATCH`

+ MAJOR 变更：不能兼容前期版本时
+ MINOR 变更：可以向后兼容，比如添加新的函数/方法/结构体字段时
+ PATH 变更：bug修复

经过以上步骤，恭喜你发布成功了。接下来可以通过使用Module来验证下自己的模块！























