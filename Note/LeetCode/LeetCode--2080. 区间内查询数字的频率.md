[2080. 区间内查询数字的频率](https://leetcode.cn/problems/range-frequency-queries/)

> 请你设计一个数据结构，它能求出给定子数组内一个给定值的 **频率** 。
>
> 子数组中一个值的 **频率** 指的是这个子数组中这个值的出现次数。
>
> 请你实现 `RangeFreqQuery` 类：
>
> - `RangeFreqQuery(int[] arr)` 用下标从 **0** 开始的整数数组 `arr` 构造一个类的实例。
> - `int query(int left, int right, int value)` 返回子数组 `arr[left...right]` 中 `value` 的 **频率** 。
>
> 一个 **子数组** 指的是数组中一段连续的元素。`arr[left...right]` 指的是 `nums` 中包含下标 `left` 和 `right` **在内** 的中间一段连续元素。

---

将 arr 的数值作为键，下标作为值存入哈希表，然后根据左右边界二分查找

```go
type RangeFreqQuery struct {
    src map[int][]int
}


func Constructor(arr []int) RangeFreqQuery {
    pos := map[int][]int{}
    for i, x := range arr {
        pos[x] = append(pos[x], i)
    }
    return RangeFreqQuery{
        src: pos,
    }
}


func (this *RangeFreqQuery) Query(left int, right int, value int) int {
    a := this.src[value]
    return sort.SearchInts(a, right + 1) - sort.SearchInts(a, left)
}

func bio(nums []int, target int) int {
    l, r := 0, len(nums)
    for l < r {
        mid := (l + r) / 2
        if nums[mid] <= target {
            l = mid + 1
        } else {
            r = mid
        }
    }
    return l
}

```

