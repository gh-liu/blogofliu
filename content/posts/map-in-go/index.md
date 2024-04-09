---
title : 'Go: Map'
date: 2021-06-22T07:52:18Z
draft : true
isCJKLanguage : true
categories:
- Development
tags:
- go
enableComment : true
---

Go里的map，就是哈希表。

1. 哈希函数
2. 解决冲突：1) 开放地址法 2)拉链法

## 创建map
1. 字面量: `kvs := map[string]string{"key": "val"}`
2. make: `kvs2 := make(map[string]string)`

## 数据结构

map 由一个`桶`的数组组成，每个`桶`里面有8个`键值对`；如果一个`桶`里超过了8个`键值对`，则会新加一个`桶`；

根据键计算的哈希可分为两部分，`低比特部分`用以选择桶，`高比特部分`用以区分键值对；

扩容：分配`两倍`桶数量大小的数组，然后`渐进式`的从旧数组复制到新数组；

```go
// runtime/map.go
type hmap struct {
	// ...
	B         uint8  // 桶的数量：2^B 个
	noverflow uint16 // 溢出桶的数量
	// ...
	buckets    unsafe.Pointer // 2^B 个桶数组指针
	oldbuckets unsafe.Pointer // 旧的桶数组指针，仅在扩容时不为nil，长度为目前桶数量的一半 2^B/2

	nevacuate  uintptr        // 小于 nevacuate 的桶都已经成功迁移到新桶数组
	// ...
}

// 一个桶
type bmap struct {
	// tophash包含了桶中每个key的哈希的高比特部分
	// 用以区分桶中的键值对
	tophash [bucketCnt]uint8
	// 后面跟着所有的键、和所有的值 key1 key2 ... value1 value2 ...
	// 还跟着溢出桶指针
}

// 由编译器生成具体的bucket结构体
//	A "bucket" is a "struct" {
//	      tophash [BUCKETSIZE]uint8
//	      keys [BUCKETSIZE]keyType
//	      elems [BUCKETSIZE]elemType
//	      overflow *bucket
//	    }
// cmd/compile/internal/reflectdata/reflect.go
func MapBucketType(t *types.Type) *types.Type {}

// runtime/map.go
type mapextra struct {
	// 桶可能有个溢出桶，由一个指针指向，储存在以下两个字段
	overflow    *[]*bmap
	oldoverflow *[]*bmap

	// 指向可用的溢出桶的指针
	nextOverflow *bmap
}

```

### 创建map的函数

```go
// runtime/map.go

// make(map[k]v, hint)
// 如果编译器决定此map或第一个桶可以放在栈上，参数 h 或 h.buckets 非 nil
func makemap(t *maptype, hint int, h *hmap) *hmap {
	// ...
	h.B = B // 2^B 满足 hint 要求
	// ...
	if h.B != 0 { // 桶数量不为零，则初始化桶数组
		h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
		// 如果预先分配了溢出桶，放在extra字段里
		if nextOverflow != nil {
			h.extra = new(mapextra)
			h.extra.nextOverflow = nextOverflow
		}
	}
}

// 初始化桶数组
// 参数 dirtyalloc 为之前分配的相同类型t和数量b的同数组，不为nil则复用
func makeBucketArray(t *maptype, b uint8, dirtyalloc unsafe.Pointer) (buckets unsafe.Pointer, nextOverflow *bmap) {
	// 计算溢出桶的数量
	if b >= 4 {}
	if dirtyalloc == nil {
		// 分配桶数组
		buckets = newarray(t.Bucket, int(nbuckets))
	} else {
		// 复用dirtyalloc桶数组: 需要清空里面的数据
	}
	// 有溢出桶，事先分配一些桶
	if base != nbuckets {}
	// ...
}

```

函数`makemap`根据hint计算出桶的数量，如果桶数量不为零，使用`makeBucketArray`初始化桶数组，为零则在后续给map里面写入数据时再来初始化数组桶(懒惰初始化)；

## 扩容/数据迁移

```go
// runtime/map.go

func hashGrow(t *maptype, h *hmap) {
	// ...
	h.B += bigger // 桶数量乘2
	h.oldbuckets = oldbuckets // 保存旧桶数组
	h.buckets = newbuckets // 分配新桶数组
	// ...
}

func growWork(t *maptype, h *hmap, bucket uintptr) {
	// 调用 evacuate 函数
	// 如果依旧处于数据迁移中，再调用 evacuate 函数一次
}

// 以桶为单位进行数据迁移 
// 参数oldbucket 表示要迁移的桶
func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
	// 获得指定的桶
	b := (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))

	if !evacuated(b) {
		var xy [2]evacDst 
		x := &xy[0]
		x.b = (*bmap)(add(h.buckets, oldbucket*uintptr(t.BucketSize)))
		// ...

		// NOTE: 有一种可能，溢出桶太多，但是没有到达溢出因子时，不会扩容，只会迁移
		if !h.sameSizeGrow() {
			y := &xy[1]
			y.b = (*bmap)(add(h.buckets, (oldbucket+newbit)*uintptr(t.BucketSize)))
			// ...
		}

		// 遍历桶，包括溢出桶
		for ; b != nil; b = b.overflow(t) {
			// 键的位置
			k := add(unsafe.Pointer(b), dataOffset)
			// 值的位置
			e := add(k, bucketCnt*uintptr(t.KeySize))
			// 分别遍历每个键/值
			for i := 0; i < bucketCnt; i, k, e = i+1, add(k, uintptr(t.KeySize)), add(e, uintptr(t.ValueSize)) {
				// ...
				// 如果是扩容，计算是否迁移到新桶(前面的桶 x或y evacDst)
				if !h.sameSizeGrow() {}
				// ...
				// 更新hash：标记迁移到哪个桶
				b.tophash[i] = evacuatedX + useY 
				// 决定迁移到哪个桶
				dst := &xy[useY]
				// 达到桶可以存放的数量（8）时，分配一个溢出桶
				if dst.i == bucketCnt {}
				// 将 top 值复制到新的桶中
				dst.b.tophash[dst.i&(bucketCnt-1)] = top
				// ... 复制键/值
				// ... 桶元素索引+1
				dst.i++
				// 更新k和e指向的键和值的位置
				dst.k = add(dst.k, uintptr(t.KeySize))
				dst.e = add(dst.e, uintptr(t.ValueSize))
			}
		}
	}
}
```

从`hashGrow`函数可知，桶数量乘2，保存旧桶数组，分配新桶数组；
旧数据迁移通过`growWork`和`evacuate`函数渐进式(对map进行修改时)完成；

`evacuate`函数会将元素从旧桶迁移到新桶，包括：决定迁移目的桶、计算哈希存放位置、迁移元素等步骤；

## 操作

增加、删除、改、查

### 赋值(写入/修改)

```go
var kv map[string]string
kv["key"] = "val"

// runtime/map.go
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	// ...
	// 根据key计算hash
	hash := t.Hasher(key, uintptr(h.hash0))
	// ...
	if h.buckets == nil { // 如果桶数组为nul，则分配一个
		h.buckets = newobject(t.bucket) // newarray(t.bucket, 1)
	}
	// ...
again:
	// 计算桶号（低位比特）
	bucket := hash & bucketMask(h.B)
	//如果处于扩容中，渐进式迁移部分数据
	if h.growing() { 
		growWork(t, h, bucket)
	}
	// 计算桶地址
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.BucketSize)))
	// 高位比特(键值对位置)
	top := tophash(hash)
	// ...
bucketloop:
	for {
		// 遍历桶中的8个键值对，比较高比特部分是否相等，再比较key是否相等（哈希冲突）
		for i := uintptr(0); i < bucketCnt; i++ { 
			// 高比特部分不等，判断是否为空
			// 为空则记录此位置，如果后续找不到top，则存在这个位置
			// 如果后续都为空，则跳过，不找了
			if b.tophash[i] != top {} 
			// ...
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize)) // 计算出对应key的位置
			// ...
			elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize)) // 计算出对应value的位置
			// ...
			goto done // NOTE: 根据hash找到了位置，返回elem的地址，也就是存放value的空间
		}

		// 当前桶没找着，溢出桶继续找
		ovf := b.overflow(t)
		if ovf == nil { break }
		b = ovf
	}
	// NOTE: 没有根据哈希找到位置，分配一个

	// 判断是否达到负载因子了，或者太多溢出桶了，满足则进行扩容，然后重新尝试一一遍
	// ...

	// 没有找到空位，也就是桶和溢出桶都满了，就再分配一个溢出桶
	if inserti == nil {
		// 再分配一个溢出桶
		newb := h.newoverflow(t, b)
		// ...
		elem = add(insertk, bucketCnt*uintptr(t.KeySize))
	}

done:
	return elem
}
```

函数`mapassign`，如果`key`不存在，则分配一个空间，否则使用原空间，存储`value`；
当出现桶满了的情况时，则分配一个新桶；

### 删除

```go
var kv map[string]string
delete(kv, "key")

// runtime/map.go
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {
		// 同map赋值类似，根据hash高比特部分找到key和value
		//...
		// 清理key
		memclrHasPointers(k, t.key.size)
		// 清理value: 两者情况，value带不带指针
		memclrHasPointers(e, t.elem.size)
		memclrNoHeapPointers(e, t.elem.size)
		// 将桶的当前key的哈希位置标记为空
		// 将map的元素数量减1
	}
}
```

同map赋值逻辑类似，遍历桶，根据hash计算出key和value所在的位置，然后清除对应的key和value，将位置标记为空；

	
### 访问或遍历

```go
var kv map[string]string
val := kv["key"]
val,exist := kv["key"]

// runtime/map.go
// 元素不存在时，则返回零值
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {}
// 第二个返回值表明key是否存在
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool) {
	// 主要逻辑同 mapaccess1 类似
}

```

访问map具体某个元素时，同赋值/删除一致，根据hash计算出key和value所在的位置；

NOTE: 在渐进式迁移数据中(`h.oldbuckets!=nil`)，先判断对应的桶是否迁移完毕(`evacuated(oldb)`)，没有则访问旧桶，否则访问新桶；

```go
kvs := make(map[string]string)
for k,v := range kvs {}

// runtime/map.go
type hiter struct {
	key         unsafe.Pointer // 当前的键
	elem        unsafe.Pointer // 当前的值
	h           *hmap // map
	B           uint8
}

// 参数it，也就是迭代器，是由编译器分配在栈上的
func mapiterinit(t *maptype, h *hmap, it *hiter) {
	// ...
	it.h = h
	it.B = h.B
	it.buckets = h.buckets
	// by design: 开始位置随机
	it.startBucket = r & bucketMask(h.B)
	it.offset = uint8(r >> h.B & (bucketCnt - 1))
	// ... 
}

func mapiternext(it *hiter) {

	// 指向桶的指针
	b := it.bptr
	// ...
next:
	if b == nil {
		// 处于数据迁移中
		if h.growing() && it.B == h.B {
			// 选择旧桶
			b = (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.BucketSize)))
			if !evacuated(b) {
			} else {
				// 如果迁移完成，选择新桶
				b = (*bmap)(add(it.buckets, bucket*uintptr(t.BucketSize)))
			}
		} else {
			// 不处于数据迁移中
			b = (*bmap)(add(it.buckets, bucket*uintptr(t.BucketSize)))
		}

		// 桶索引递增
		bucket++
	}
	for ; i < bucketCnt; i++ { // 以桶为单位遍历
		// 计算偏移
		offi := (i + it.offset) & (bucketCnt - 1)
		// 根据偏移计算出key在桶里的位置
		k := add(unsafe.Pointer(b), dataOffset+uintptr(offi)*uintptr(t.keysize))
		// 根据偏移计算出value在桶里的位置
		e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+uintptr(offi)*uintptr(t.elemsize))

		// 赋值迭代器当前的key/value
		it.key = k
		it.elem = e
	}
	// 溢出桶
	b = b.overflow(t)
	i = 0
	goto next
}
```

NOTE: 以桶为单位进行遍历，如果处于渐进式数据迁移中，如果旧桶未完成迁移，则选择旧桶，否则使用新桶；

## 更多阅读

[Hash Table](https://en.wikipedia.org/wiki/Hash_table)
