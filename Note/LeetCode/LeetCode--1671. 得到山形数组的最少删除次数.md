[1671. 得到山形数组的最少删除次数](https://leetcode.cn/problems/minimum-number-of-removals-to-make-mountain-array/)

> 我们定义 `arr` 是 **山形数组** 当且仅当它满足：
>
> - `arr.length >= 3`
>
> - 存在某个下标 `i`（从 0 开始）满足 `0 < i < arr.length - 1`
>
>    且：
>
>   - `arr[0] < arr[1] < ... < arr[i - 1] < arr[i]`
>   - `arr[i] > arr[i + 1] > ... > arr[arr.length - 1]`
>
> 给你整数数组 `nums` ，请你返回将 `nums` 变成 **山形状数组** 的 **最少** 删除次数。

---

两个最长上升子序列问题

```go
func minimumMountainRemovals(nums []int) int {
    n := len(nums)
    fl, fr := make([]int, n), make([]int, n)
    ans := 0
    for i := 0; i < n; i ++ {
        fl[i] = 1
        for j := 0; j <= i; j ++ {
            if nums[j] < nums[i] {
                fl[i] = max(fl[i], fl[j] + 1)
            }
        }
    }
    for i := n - 1; i >= 0; i -- {
        fr[i] = 1
        for j := n - 1; j >= i; j -- {
            if nums[j] < nums[i] {
                fr[i] = max(fr[i], fr[j] + 1)
            }
        }
    }
    for i := 0; i < n; i ++ {
        if fl[i] > 1 && fr[i] > 1 {
            ans = max(ans, fr[i] + fl[i] - 1)
        }
    }
    return n - ans
}
```

