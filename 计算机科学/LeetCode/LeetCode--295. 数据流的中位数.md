[295. 数据流的中位数](https://leetcode.cn/problems/find-median-from-data-stream/)

> **中位数**是有序整数列表中的中间值。如果列表的大小是偶数，则没有中间值，中位数是两个中间值的平均值。
>
> - 例如 `arr = [2,3,4]` 的中位数是 `3` 。
> - 例如 `arr = [2,3]` 的中位数是 `(2 + 3) / 2 = 2.5` 。
>
> 实现 MedianFinder 类:
>
> - `MedianFinder() `初始化 `MedianFinder` 对象。
> - `void addNum(int num)` 将数据流中的整数 `num` 添加到数据结构中。
> - `double findMedian()` 返回到目前为止所有元素的中位数。与实际答案相差 `10-5` 以内的答案将被接受。

---

面试出这种题是真相似了😫😫😫

构造两个堆，大根堆和小根堆，内部的数字数量差不超过1，如果超过，则需要进行平衡调整。

```go
type MaxHeap []int
type MinHeap []int

type MedianFinder struct {
	*MaxHeap
	*MinHeap
}

func (h MaxHeap) Less(i, j int) bool {
	return h[j] < h[i]
}

func (h MinHeap) Less(i, j int) bool {
	return h[j] > h[i]
}

func (h MinHeap) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func (h MaxHeap) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func (h MinHeap) Len() int {
	return len(h)
}

func (h MaxHeap) Len() int {
	return len(h)
}

func (h *MaxHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *MinHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *MaxHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

func (h *MinHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}


func Constructor() MedianFinder {
	ah := &MaxHeap{}
	heap.Init(ah)
	ih := &MinHeap{}
	heap.Init(ih)
	return MedianFinder{
		MaxHeap: ah,
		MinHeap: ih,
	}
}


func (this *MedianFinder) AddNum(num int)  {
    switch this.MaxHeap.Len() - this.MinHeap.Len(){
        case 0:
        if this.MaxHeap.Len() == 0 || num < (*this.MaxHeap)[0] {
            heap.Push(this.MaxHeap, num)
        } else {
            heap.Push(this.MinHeap, num)
        }
        break

        case -1:
        if this.MaxHeap.Len() == 0 || num < (*this.MaxHeap)[0] {
            heap.Push(this.MaxHeap, num)
        } else {
            heap.Push(this.MinHeap, num)
            heap.Push(this.MaxHeap, heap.Pop(this.MinHeap))
        }
        break
    
        case 1:
        if num > (*this.MaxHeap)[0] {
            heap.Push(this.MinHeap, num)
        } else {
            heap.Push(this.MaxHeap, num)
            heap.Push(this.MinHeap, heap.Pop(this.MaxHeap))
        }
        break
    }

}


func (this *MedianFinder) FindMedian() float64 {
	switch this.MaxHeap.Len() - this.MinHeap.Len() {
	case 0:
		return float64((*this.MaxHeap)[0] + (*this.MinHeap)[0]) / 2.0
	case 1:
		return float64((*this.MaxHeap)[0])
	default:
		return float64((*this.MinHeap)[0])
	}
}

```

