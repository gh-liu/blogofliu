---
title: "Go: Build Constraints"
date: 2021-08-15T11:38:19Z
draft : false
isCJKLanguage : true
categories:
- Development
tags:
- go
enableComment : true
---

## Build Constraints 构建约束

> 在开发Go包时，如果依赖特定平台或处理器特定功能时，往往需要提供对应的实现。

> go/build包提供了一种命名标记的约定，go工具链允许go包自定义需要被编译进去的go文件，每个文件要么在编译中，要么不在。

> 构建约束(build constraints)也称作构建标记(build tag)

命名标记的约定有两种方式:
1. 文件内首行注释
2. 文件名格式

### 文件内首行注释

```go
//go:build XXXXXX
```

文件**首行**注释中的**go:build XXXXXX**标记会被解析为构建约束。
注意，注释后没有空格；且后接一个空白行，也就是构建约束出现在package语句之前。

```text
XXXXXX 列出了在什么情况下，该文件被编译进去。
约束XXXXXX 会被计算为一个表达式，由||, &&, !和括号组成。
```

比如:
```go
//go:build (linux && 386) || (darwin && !cgo)

// 这个文件在满足 "linux" 和 "386"，或满足"darwin"并且"cgo"不满足时，会被编译进去。
```


在一个构建中，下列词可以作为约束:
1. 目标操作系统: 环境变量GOOS
2. 目标架构： 环境变量GOARCH
3. 使用的编译器: "gc" 或 "gccgo"
4. go 版本号: 比如"go1.1"，表示从哪个版本号开始
5. 任意其他的标签: 在build时，使用-tags标志传递给go工具链

### 文件名格式

构建约束也可以是文件名的一部分，比如 source_windows.go 会在目标操作系统为windows时被编译。

文件名匹配下列任意模式: (比如: source_windows_amd64.go)
```text
*_GOOS
*_GOARCH
*_GOOS_GOARCH
```

GOOS和GOARCH代表任意已知的操作系统与架构，然后文件会被视为有一个隐式的构建约束。

Go versions 1.16 and earlier used a different syntax for build constraints, with a "// +build" prefix. The gofmt command will add an equivalent //go:build constraint when encountering the older syntax.

Go1.16与之前的版本对约束使用的是另一种不同的语法，使用了一个"// +build"前缀，1.17对构建约束进行了[修改](https://go.googlesource.com/proposal/+/master/design/draft-gobuild.md)。
gofmt命令会在遇到旧的语法时，添加一个等价的//go:build约束。

### 在build时使用约束

目标操作系统和目标架构可以通过设置环境变量来完成：
```bash
GOOS=windows GOARCH=amd64 go build main.go
```

自定义约束
```bash
go build -tags tag1,tag2
```


[1]: <https://pkg.go.dev/go/build#hdr-Build_Constraints> "Build_Constraints"
[2]: <https://pkg.go.dev/cmd/go#hdr-Build_constraints> ""
[3]: <https://blog.csdn.net/eddycjy/article/details/118981411> ""
[4]: <https://dave.cheney.net/2013/10/12/how-to-use-conditional-compilation-with-the-go-build-tool> ""
[5]: <https://www.digitalocean.com/community/tutorials/customizing-go-binaries-with-build-tags> ""

