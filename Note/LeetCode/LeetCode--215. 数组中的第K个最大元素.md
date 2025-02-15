[215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

>给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。
>
>请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。
>
>你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

----

## 堆

```go
func findKthLargest(nums []int, k int) int {
	h := &Heap{}
	heap.Init(h)
	for i := 0; i < len(nums); i++ {
		heap.Push(h, nums[i])
	}
	for h.Len() > k {
		heap.Pop(h)
	}
	return heap.Pop(h).(int)
}

type Heap []int

func (h *Heap) Len() int {
	return len(*h)
}

func (h *Heap) Less(i, j int) bool {
	return (*h)[i] < (*h)[j]
}

func (h *Heap) Swap(i, j int) {
	(*h)[i], (*h)[j] = (*h)[j], (*h)[i]
}

func (h *Heap) Push(x any) {
	*h = append(*h, x.(int))
}

func (h *Heap) Pop() any {
	Num := (*h)[len(*h)-1]
	*h = (*h)[:len(*h)-1]
	return Num
}

```

----



## 快选

```go
func quickselect(nums []int, l int, r int, k int) int {
    if l == r {
        return nums[k]
    }
    partition := nums[l]
    i := l - 1
    j := r + 1
    for i < j {
        for i++;nums[i]<partition;i++{}
        for j--;nums[j]>partition;j--{}
        if i < j {
            swap(&nums[i], &nums[j])
        }
    }
    if k <= j {
        return quickselect(nums, l , j, k)
    } else {
        return quickselect(nums, j + 1, r, k)
    }
}

func swap(a *int, b *int) {
    x := *a
    *a = *b
    *b = x
}


func findKthLargest(nums []int, k int) int {
    return quickselect(nums, 0, len(nums) - 1, len(nums) -k)
}
```

