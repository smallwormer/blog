---
stitle: "go 数据类型"
date: 2018-12-30T17:12:17+08:00
draft: true
---



## 数组

类型[n]T 是一个有n个类型为T的值的数组

表达式为

​	var a[10]int  表示 是a是一个 有十个整数的数组

1. 数组的长度是其类型的一部分，因此数组不能改变大小。



举个栗子：

```go
package main

import "fmt"

func main() {
	var a [3]int
	a[0] = 1
	a[1] = 2
	a[2] = 3
	fmt.Println(a[0], a[1])
	fmt.Println(a)

}
```



输出结果为

```
1 2
[1 2 3]
```



##  slice

slice 是指向一个序列的值，并且包含了长度信息

[]T 是一个元素类型为T 的slice

```go
package main

import "fmt"


func main(){
	p := []int{2,3,4,5,6,8,9}
	fmt.Println("p == ", p)
	for i := 0;i < len(p);i++ {
		fmt.Printf("p[%d] == %d\n", i, p[i])
	}
}
```



输出结果为：

```go
p ==  [2 3 4 5 6 8 9]
p[0] == 2
p[1] == 3
p[2] == 4
p[3] == 5
p[4] == 6
p[5] == 8
p[6] == 9
```



### 对slice进行切片

1. s[m:n] 表示 从m 到n-1 的slice元素，包含两端
2. s[m:m] 是空元素
3. s[m:m+1]是表示有一个元素  

```go
package main

import "fmt"


func main(){
	p := []int{2,3,4,5,6,8,9}
	fmt.Println("p == ", p)

	fmt.Println("p[1:4] ==", p[1:4])
	fmt.Println("p[:3]==",p[:3])

	fmt.Println("p[4:] ==", p[4:])
}
```

输出结果为：

```sh
p ==  [2 3 4 5 6 8 9]
p[1:4] == [3 4 5]
p[:3]== [2 3 4]
p[4:] == [6 8 9]
```



##  slice 切片的len cap

 我们知道早GO语言中，数组的长度是不可以变的，那么为了更灵活的处理数据，才有了slice,可以理解为动态数组。但是slice并不是真正意义上的动态数组，而是一个引用类型。slice总是指向一个底层array,slice的声明可以向array一样，只是不需要长度。



```go
// 这里声明了一个保存int的slice islice
var islice []int

// 声明一个长度为10的int数组
var iarry [10]int

// 创建一个保存int的slice,其中len长度为5， 容量cap为10.这里指定了容量以及长度，非动态
slice2 := make([]int,5,10)
```



### 长度len() 以及 容量cap()

len 表示 slice的长度，表示它包含的元素的个数

cap 表示从它的第一个元素开始数，到其底层数组元素末尾的个数

```go
//这里表示 为长度为5的int slice 开辟了容量为10的内存
slice3 := make([]int,5,10)

// 这样就不会开始时固定内存的大小。建议是直接指定cap大小，这样就避免多次改变导致多次重新改变cap分配空间带来的不必要的开销
slice4 := make([]int,5)
```



### append()

```go
package main

import "fmt"

func main() {
	var s []int
	printSlice(s)

	// append works on nil slices.
	s = append(s, 0)
	printSlice(s)

	// The slice grows as needed.
	s = append(s, 1)
	printSlice(s)

	// We can add more than one element at a time.
	s = append(s, 2, 3, 4)
	printSlice(s)
	s = append(s, 5, 6, 7, 8, 9, 10)
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```



输出结果：

```
len=0 cap=0 []
len=1 cap=2 [0]
len=2 cap=2 [0 1]
len=5 cap=8 [0 1 2 3 4]
len=11 cap=16 [0 1 2 3 4 5 6 7 8 9 10]
```



可以看到 cap() 有了变化.

### append的运作机制

在对slice进行append等操作时，可能会造成slice的自动扩容。在扩容时的大小增长规则是:

* 如果新的slice大小是当前大小的2倍以上，则大小增长为新大小
* 否则：
  * 如果当前slice大小小于1024，则按照每次2倍进行增长，否则每次按照当前大小的1.25倍增长，直到增长的大小超过或者等于新大小
* append的实现只是简单的在内存中将旧的slice复制给新的slice

go 代码：/usr/local/go/src/runtime/slice.go

```go
func growslice(et *_type, old slice, cap int) slice {
	if raceenabled {
		callerpc := getcallerpc()
		racereadrangepc(old.array, uintptr(old.len*int(et.size)), callerpc, funcPC(growslice))
	}
	if msanenabled {
		msanread(old.array, uintptr(old.len*int(et.size)))
	}

	if et.size == 0 {
		if cap < old.cap {
			panic(errorString("growslice: cap out of range"))
		}
		// append should not create a slice with nil pointer but non-zero len.
		// We assume that append doesn't need to preserve old.array in this case.
		return slice{unsafe.Pointer(&zerobase), old.len, cap}
	}

	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
```



