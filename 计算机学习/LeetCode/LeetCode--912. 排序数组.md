[912. 排序数组](https://leetcode.cn/problems/sort-an-array/)

>给你一个整数数组 `nums`，请你将该数组升序排列。
>
>你必须在 **不使用任何内置函数** 的情况下解决问题，时间复杂度为 `O(nlog(n))`，并且空间复杂度尽可能小。

---

## 正文

### 手撕快排

若基准元素选择中间元素，在本题中会无限递归，若选择l，则会时间过长，所以最好的选择就是随机数选择，这样可以将时间优化到最低。

```go
func sortArray(nums []int) []int {
	quicksort(nums, 0, len(nums)-1)
	return nums
}

func quicksort(nums []int, l int, r int) {
	if l >= r {
		return
	}
	pivotIndex := rand.Intn(r-l+1) + l
	swap(&nums[pivotIndex], &nums[(l+r)>>1])
	mid := (l + r) >> 1
	pivot := nums[mid]
	i, j := l-1, r+1
	for i < j {
		for i++; nums[i] < pivot; i++ {}
       for j--; nums[j] > pivot; j-- {}
		if i < j {
			swap(&nums[i], &nums[j])
		}
	}
	quicksort(nums, l, j)
	quicksort(nums, j+1, r)
}

func swap(i, j *int) {
	x := *i
	*i = *j
	*j = x
}
```

----

### 二刷，堆排序

直接手写down，pop，up，push

```go
type Heap []int

func sortArray(nums []int) []int {
	h := Heap{}
	for i := 0; i < len(nums); i++ {
		h.Push(nums[i])
	}
	var ans []int
	for i := 0; i < len(nums); i++ {
		ans = append(ans, h.Pop())
	}
	return ans
}

func (h *Heap) Push(num int) {
	*h = append(*h, num)
	(*h).Up(len(*h) - 1)
}

func (h *Heap) Up(idx int) {
	if idx == 0 {
		return
	}
	if (*h)[idx] < (*h)[idx/2] {
		(*h)[idx], (*h)[idx/2] = (*h)[idx/2], (*h)[idx]
		(*h).Up(idx / 2)
	}
}

func (h *Heap) Down(idx int) {
	m := idx
	if len(*h) > idx*2 && (*h)[m] > (*h)[idx*2] {
		m = idx * 2
	}
	if len(*h) > idx*2+1 && (*h)[m] > (*h)[idx*2+1] {
		m = idx*2 + 1
	}
	if m != idx {
		(*h)[m], (*h)[idx] = (*h)[idx], (*h)[m]
		(*h).Down(m)
	}
	return
}

func (h *Heap) Pop() int {
	if len(*h) == 0 {
		return 0
	}
	res := (*h)[0]
	(*h)[0], (*h)[len(*h)-1] = (*h)[len(*h)-1], (*h)[0]
	(*h) = (*h)[:len(*h)-1]
	h.Down(0)
	return res
}
```

### 三刷，手撕归并

```go
func sortArray(nums []int) []int {
    MergeSort(nums, 0, len(nums) - 1)
    return nums
}


func MergeSort(nums []int, left, right int) {
    if left >= right {
        return
    }
    mid := (left + right) / 2

    MergeSort(nums, left, mid)
    MergeSort(nums, mid + 1, right)

    tmp := make([]int, 0)
    i, j := left, mid + 1
    for i <= mid && j <= right {
        if nums[i] < nums[j] {
            tmp = append(tmp, nums[i])
            i ++
        } else {
            tmp = append(tmp, nums[j])
            j ++
        }
    }
    for j <= right {
        tmp = append(tmp, nums[j])
        j ++
    }
    for i <= mid {
        tmp = append(tmp, nums[i])
        i ++
    }
    if len(tmp) != right - left + 1 {
        return
    }
    k, m := left, 0
    for k <= right {
        nums[k] = tmp[m]
        k ++
        m ++
    }
}
```

