---
title : 'Go: String'
date: 2021-04-15T07:52:18Z
draft : false
isCJKLanguage : true
categories:
- Development
tags:
- go
enableComment : true
---

### 创建字符串

双引号、反引号两种创建方式；前者称作`interpreted string`，后者称作`raw string`；

可以从名字中看出两者不同：反引号创建的字符串可以包含任意字符，而双引号不能包含未转义的双引号和多行；

```go
str1 := "this is a string"

str2 := `this is "another"
string`
```

### 字符串 = 字符数组

字符串，可以看作一个字符数组；

一个字符由若干个字节组成，也就可以看作：一个指向内存区域的指针，一个长度；
```go
type stringStruct struct {
    // 指向内存区域的指针
	str unsafe.Pointer
    // 长度
	len int
}
```

NOTE: Go中字符串使用UTF-8编码，一个字符可能由多个字节存储；

### 只读特性

```go
// main.go
package main

func main() {
	str := "this_is_a_string"
	println(str)
}
```

```bash
go tool compile -S string.go |grep "this_is_a_string"

# 可以看到如下输出：字符串被分配在了 SRODATA 只读区域
# go:string."this_is_a_string" SRODATA dupok size=16
#         0x0000 74 68 69 73 5f 69 73 5f 61 5f 73 74 72 69 6e 67  this_is_a_string
```

由于Go中的字符串是只读的，不能直接操作；
那么需要借助字节数组`byte[]`达到操作的目的：
1. 拷贝字符串到内存中
2. 将内存中的字符串转换成字节数组`byte[]`
3. 操作字节数组`byte[]`
4. 将字节数组`byte[]`转换成字符串

NOTE: 
为什么不能在字符串原位置进行修改：

安全性: 不可变性有助于避免一些安全问题，比如越界访问问题，并发数据竞争问题等

### 类型转换 string <-> []byte

```go
package main

func main() {
	str := "this_is_a_string"
	b := []byte(str) // string -> []byte
	println(string(b)) // []byte -> string
}
```
对代码进行编译，可以看出分别调用了`stringtoslicebyte`,`slicebytetostring`
```bash
go tool compile -S string.go | grep runtime
0x0028 00040 (string.go:5)  CALL    runtime.stringtoslicebyte(SB)
0x0038 00056 (string.go:6)  CALL    runtime.slicebytetostring(SB)
```

#### string -> []byte
```go
// runtime/string.go
func stringtoslicebyte(buf *tmpBuf, s string) []byte {
	var b []byte
	if buf != nil && len(s) <= len(buf) {
		*buf = tmpBuf{}
		b = buf[:len(s)]
	} else {
		b = rawbyteslice(len(s))
	}
	copy(b, s)
	return b
}
```

函数内部分配了一个字节数组，然后使用copy函数复制数据进入新的字节数组，然后返回此字节数组；

第一个参数buf的作用：当字符串小于32(tempBuf底层数组长度)时，使用buf的空间（可能分配在栈上），而不是重新分配空间；

#### []byte -> string

```go
// runtime/string.go
func slicebytetostringtmp(ptr *byte, n int) string {
	// ...
	return unsafe.String(ptr, n)
}
```
指针指向的内存区域，加上长度，就是字符串；

### 操作：以拼接为例子

根据被拼接的字符串数量选择不同的逻辑:
- 如果小于或者等于 5 个，调用 concatstring{2,3,4,5} 等一系列函数，最终也是调用 runtime.concatstrings；
- 如果超过 5 个，直接调用 runtime.concatstrings 传入一个数组切片；

```go
// runtime/string.go

// 实现了字符串的拼接操作 x + y + z...
func concatstrings(buf *tmpBuf, a []string) string {}

func concatstring2(buf *tmpBuf, a0, a1 string) string {
	return concatstrings(buf, []string{a0, a1})
}

func concatstring3(buf *tmpBuf, a0, a1, a2 string) string {
	return concatstrings(buf, []string{a0, a1, a2})
}

func concatstring4(buf *tmpBuf, a0, a1, a2, a3 string) string {
	return concatstrings(buf, []string{a0, a1, a2, a3})
}

func concatstring5(buf *tmpBuf, a0, a1, a2, a3, a4 string) string {
	return concatstrings(buf, []string{a0, a1, a2, a3, a4})
}
```

可以从`concatstrings`看出，字符串拼接的时候，发生了拷贝，也就是说最终得到的字符串存在于新的一片内存空间；

### 拼接性能优化

`+` 运算符 vs `strings.Builder 或 fmt.Sprintf`

```go
package main

import (
	"strings"
	"testing"
)

func contactByAddSign(strs []string) string {
	var result string
	for _, str := range strs {
		result += str
	}
	return result
}

func contactByBuilder(strs []string) string {
	var builder strings.Builder
	for _, str := range strs {
		builder.WriteString(str)
	}
	return builder.String()
}

var strs = []string{
	"2111112",
	"3111113",
	"4111114",
	"5111115",
	"6111116",
	"7111117",
	"8111118",
	"9111119",
	"101111110",
	"111111111",
	"121111112",
	"131111113",
	"141111114",
	"151111115",
	"161111116",
	"171111117",
	"181111118",
	"191111119",
	"201111120",
	"211111121",
	"221111122",
	"231111123",
	"241111124",
	"251111125",
}

func BenchmarkContactString(b *testing.B) {
	for i := 0; i < b.N; i++ {
		contactByAddSign(strs)
	}
}
func BenchmarkContactString2(b *testing.B) {
	for i := 0; i < b.N; i++ {
		contactByBuilder(strs)
	}
}
```

```bash
go test -bench=. -benchmem ./string_test.go
# 可以得出如下结果：
# BenchmarkContactString-8         1733986               657.2 ns/op          2536 B/op         23 allocs/op
# BenchmarkContactString2-8        6335858               189.8 ns/op           504 B/op          6 allocs/op
```

根据结果可知，使用`strings.Builder`会更快，更少使用内存。
