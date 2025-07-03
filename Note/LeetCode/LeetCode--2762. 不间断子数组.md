[2762. 不间断子数组](https://leetcode.cn/problems/continuous-subarrays/)

> 给你一个下标从 **0** 开始的整数数组 `nums` 。`nums` 的一个子数组如果满足以下条件，那么它是 **不间断** 的：
>
> - `i`，`i + 1` ，...，`j` 表示子数组中的下标。对于所有满足 `i <= i1, i2 <= j` 的下标对，都有 `0 <= |nums[i1] - nums[i2]| <= 2` 。
>
> 请你返回 **不间断** 子数组的总数目。
>
> 子数组是一个数组中一段连续 **非空** 的元素序列。

---

在一个子数组中，就是说最大值和最小值的差不能超过 2，知道这一点，依旧滑窗，哈希表存储当前窗口的数值，所以需要用 delete 去删除为 0 的元素

```go
func continuousSubarrays(nums []int) int64 {
    cnt := make(map[int]int)
    l := 0 
    var ans int64 = 0
    for r, x := range nums {
        cnt[x] ++
        for {
            mx, mn := x, x
            for k := range cnt {
                mx = max(mx, k)
                mn = min(mn, k)
            }
            if mx - mn <= 2 {
                break
            }
            out := nums[l]
            cnt[out] --
            if cnt[out] == 0 {
                delete(cnt, out)
            }
            l ++
        }
        ans += int64(r - l + 1)
    }
    return ans
}
```

