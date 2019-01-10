---
ftitle: "Go_snip"
date: 2019-01-07T16:21:23+08:00
draft: true
---



# go一般性记录

### range

```go
package main

import "fmt"

var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}

```



输出为：

```
2**0 = 1
2**1 = 2
2**2 = 4
2**3 = 8
2**4 = 16
2**5 = 32
2**6 = 64
2**7 = 128
```



range 说明：

for循环的range 形式可以遍历slice 或者映射。

当使用for循环遍历时，每次迭代会返回两个值。第一个值为当前元素的下表，第二个值为该下标所对应元素的一份副本。

如果只需要获取 值，则将下标或者值 赋予 _ 来忽略它。

```go
package main

import "fmt"

func main() {
	pow := make([]int, 8)
	for i := range pow {
		pow[i] = 1 << uint(i) // == 2**i
	}
	for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
}

输出：
1
2
4
8
16
32
64
128
```



若是只需要索引，则去掉 ", value" 的部分即可

```go
package main

import "fmt"

func main() {
	pow := make([]int, 8)
	for i := range pow {
		pow[i] = 1 << uint(i) // == 2**i
	}
	for i := range pow {
		fmt.Printf("%d\n", i)
	}
}

输出：
0
1
2
3
4
5
6
7
```

