---
title : 'Go: Slice'
date: 2021-05-06T07:52:18Z
draft : false
isCJKLanguage : true
categories:
- Development
tags:
- go
enableComment : true
---

切片像数组，但切片的长度是动态的；

## 创建切片

1. indxe: 通过下标的方式获得数组或者切片的一部分； `arr[0:3] or slice[0:3]`
2. literal: 使用字面量初始化新的切片； `slice := []int{1, 2, 3}`
3. make: 使用关键字 make 创建切片： `slice := make([]int, 10)`

NOTE:
1. 下标初始化切片不会拷贝原数组或者原切片中的数据，它只会创建一个指向原数组的切片结构体，所以修改新切片的数据也会修改原切片
2. 根据切片中的元素数量对底层数组的大小进行推断并创建一个数组，将这些字面量元素存储到初始化的数组中；对数组进行`array[:]`操作即可等到切片
3. 根据切片的大小和容量判断是否需要逃逸到堆上，需要则调用runtime.makeslice，否则在栈上初始化底层数组同时获得切片

## 切片 = 指向结构体的指针

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

可以看出，切片由指向内存区域的一个指针、长度、容量组成。

实际上：当切片长度不足时就会触发扩容；
切片指向的底层数组可能会发生变化，不过在上层看来切片是没有变化的，上层只需要与切片打交道，不需要关心底层数组的变化。

## 相关内置函数

```go
// builtin/builtin.go

// 由编译器处理,返回len字段
func len(v Type) int

// 由编译器处理,返回cap字段
func cap(v Type) int

// 从src切片复制元素到dst切片
// 复制元素数量取 dst 和 src 两者长度中较小者
func copy(dst, src []Type) int

// 追加元素到slice切片，并返回新切片
// 容量不够时候，底层数组会进行扩容
func append(slice []Type, elems ...Type) []Type
```

## grow slice

切片的扩容操作，会分配一个新的底层数组；

```go
// runtime/slice.go

// 参数为：
// oldPtr 指向原底层数组的指针
// newLen 新长度 = oldCap + num
// oldCap 旧容量
// num 新添加的元素数量
// et 元素类型
// 返回值：切片，包含新底层数组指针，新长度，新容量
func growslice(oldPtr unsafe.Pointer, newLen, oldCap, num int, et *_type) slice {}
```

扩容流程：
1. 如果要求的 newLen 大于 oldCap 的两倍，则newCap = newLen；否则

2. (1) 如果 oldCap 小于256，newCap = oldCap * 2; 否则，(2) 循环：newCap += (newCap + 3*256) / 4 直到 newCap > newLen, 增加速度缓慢递减 1.63 -> 1.44 -> 1.35 -> 1.30；

3. 然后，根据newCap分配内存

4. 然后，从旧指针处，移动原来元素的内容，到新指针指向的内存处

总结：

- 如果期望容量大于当前容量的两倍就会使用期望容量；
- 如果原切片的长度小于 256 就会将容量翻倍；
- 如果当前切片的长度大于 256 就会增加速度逐步递减 1.63 -> 1.44 -> 1.35 -> 1.30 的容量，直到新容量大于期望容量；

NOTE: 原实现是小于1024翻倍，大于1024则1.25倍

## 性能问题

1. 切片扩容：涉及到内存重新分配、数据复制；所以尽量避免扩容，在知道切片最终大小的情况下，创建时即制定容量；
2. 固定大小时，使用数组
3. 如果由大量短期的切片，使用`sync.Pool`进行切片的池化，减轻GC压力

## 更多阅读

[inside-a-go-slice](https://chidiwilliams.com/posts/inside-a-go-slice)

[go blog: slices]( https://go.dev/blog/slices )

[go blog: slices-intro]( https://go.dev/blog/slices-intro )

[array-vs-slice-accessing-speed]( https://stackoverflow.com/questions/30525184/array-vs-slice-accessing-speed )
