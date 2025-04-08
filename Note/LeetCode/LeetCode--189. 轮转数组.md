[189. 轮转数组](https://leetcode.cn/problems/rotate-array/)

> 给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

---

先将整体数组反转，再以k为分割线，分别反转两侧数组，就可以得到我们的答案。

```go
func rotate(nums []int, k int) {
    n := len(nums)
    k = k % n
    reverse(0, n - 1, nums)
    reverse(0, k - 1, nums)
    reverse(k, n - 1, nums)
}

func reverse(i, j int, nums []int) {
    for i < j {
        nums[i], nums[j] = nums[j], nums[i]
        i ++
        j --
    }
}
```