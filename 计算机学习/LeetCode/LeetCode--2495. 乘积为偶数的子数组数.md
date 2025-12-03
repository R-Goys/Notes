[2495. 乘积为偶数的子数组数](https://leetcode.cn/problems/number-of-subarrays-having-even-product/)

> 给定一个整数数组 `nums`，返回*具有偶数乘积的* `nums` *子数组的数目*。

---

秒了，一旦遇见偶数，直接加入结果就行，此处通过 l 计算比较方便。

```go
func evenProduct(nums []int) int64 {
    var ans int64 = 0
    l := 0
    for r, x := range nums {
        if x % 2 == 0 {
            l = r + 1
        }
        ans += int64(l)
    }
    return ans
}
```

