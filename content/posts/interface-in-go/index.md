---
title : 'Go: Interface'
date: 2022-05-16T07:52:18Z
draft : true
isCJKLanguage : true
categories:
- Development
tags:
- go
enableComment : true
---

## 数据结构

两种实现：非空接口、空接口

非空接口包含`数据指针`，`数据的类型指针`，`方法列表`；
空接口包含  `数据指针`，`数据的类型指针`；

```go
// runtime/runtime2.go

// 1. 带方法的interface{}
type iface struct {
	tab  *itab
	data unsafe.Pointer // 数据指针
}

type itab struct {
	inter *interfacetype
	_type *_type // 数据的类型指针
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}

// 2. 空(不带方法)interface{}
type eface struct {
	_type *_type // 数据的类型指针
	data  unsafe.Pointer // 数据指针
}
```

## 含有nil值的接口不等于nil

```go
package main

import "fmt"

func main() {
	var a interface{}
	fmt.Printf("a == nil is %t\n", a == nil)
	var b interface{}
	var p *int = nil
	b = p
	fmt.Printf("b == nil is %t\n", b == nil)
}
```

## 动态分发

## 接口组合

## 类型断言

## 更多阅读

[ How To Use Go Interfaces ](https://blog.chewxy.com/2018/03/18/golang-interfaces/)
