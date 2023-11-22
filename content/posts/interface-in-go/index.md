---
title : 'Go: Interface'
date: 2022-05-16T07:52:18Z
draft : false
isCJKLanguage : true
categories:
- Development
tags:
- go
enableComment : true
---

## 数据结构

```go
// runtime/runtime2.go

// 1. 带方法的interface{}
type iface struct {
	tab  *itab
	data unsafe.Pointer
}

type itab struct {
	inter *interfacetype
	_type *_type
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}

// 2. 空(不带方法)interface{}
type eface struct {
	_type *_type
	data  unsafe.Pointer
}
```

## 更多阅读

[ How To Use Go Interfaces ](https://blog.chewxy.com/2018/03/18/golang-interfaces/)
