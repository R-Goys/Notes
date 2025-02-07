[例题链接-前k个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/submissions/596342618/?envType=study-plan-v2&envId=top-100-liked)

----

## 前言

以前都是用的C++写算法题，最近也想熟悉一下golang的数据结构，故来一篇题解+堆分析。



## 正文

这里重点不在分析题目，在于golang中的 `container/heap`

对于内部实现逻辑有兴趣的可以去看看源码。

这里先给出题解的代码

```go	
package main

import (
	"container/heap"
	"fmt"
)

// IHeap 是一个最小堆的实现
type IHeap [][2]int

func (h IHeap) Len() int {
	return len(h)
}

func (h IHeap) Less(i, j int) bool {
	return h[i][1] < h[j][1]
}
func (h IHeap) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

// Push 方法将元素添加到堆中
func (h *IHeap) Push(x interface{}) {
	*h = append(*h, x.([2]int))
}

// Pop 方法移除并返回堆顶元素
func (h *IHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

// topKFrequent 函数找到数组中出现频率最高的 k 个元素
func topKFrequent(nums []int, k int) []int {
	// 统计每个数字的出现频率
	m := map[int]int{}
	for _, num := range nums {
		m[num]++
	}

	// 创建最小堆
	h := &IHeap{}
	heap.Init(h)

	// 将元素推入堆并维护堆的大小
	for key, value := range m {
		heap.Push(h, [2]int{key, value})
		if h.Len() > k {
			heap.Pop(h)
		}
	}

	// 从堆中提取结果
	ret := make([]int, k)
	for i := 0; i < k; i++ {
		ret[k-i-1] = heap.Pop(h).([2]int)[0]
	}
	return ret
}

func main() {
	nums := []int{1, 1, 216, 216, 216, 216, 216, 216, 6, 1, 2, 2, 3, 9, 9, 5, 6, 0, 6, 6, 9, 4, 5, 12, 6, 459, 15, 15, 216, 26, 15, 115, 15}
	k := 5
	fmt.Println(topKFrequent(nums, k))
}
```

### 1. 结构定义

这部分定义了我们的堆中元素的基本结构，每个元素有两部分组成，这也令go中的堆的元素更加灵活，可以支持很多数据结构。

```go
type IHeap [][2]int
```

### 2. Len()方法

首先，需要实现我们的Len方法，实现这个方法的目的是，他将会在之后函数内部的Down/Up方法所调用，具有重要的作用(这部分在源码里面)

```go	
func (h IHeap) Len() int {
	return len(h)
}
```

### 3. Less()方法

Less方法的定义主要是实现了堆内部的比较器，也就是排序原则，就是大根堆和小根堆的区别

```go
func (h IHeap) Less(i, j int) bool {
	return h[i][1] < h[j][1]
}
```

### 4. Swap()方法

这部分也是主要用于Down和Up内部调用，表示利用传入的下标来进行元素位置的交换

```go
func (h IHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}
```

### 5. Push()方法

push方法也是用于内部的push函数的调用，此处不需要进行Down或者Up的操作，因为内部的Push函数已经为你准备好了。

```go
func (h *IHeap) Push(x interface{}) {
	*h = append(*h, x.([2]int))
}
```

### 6. Pop()方法

移除堆顶元素的方法，同样用于内部调用

```go
func (h *IHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}
```



## 结语

以上提到方法都需要我们自己定义一个`堆`,并实现Heap的接口，才能对heap的函数进行调用，从而实现堆的效果。

总的来说，go的堆虽然实现比较繁琐，但是管理起来却比较灵活，其实比起c++里面的stl，go里面的container/heap让我更有写的欲望吧...😋