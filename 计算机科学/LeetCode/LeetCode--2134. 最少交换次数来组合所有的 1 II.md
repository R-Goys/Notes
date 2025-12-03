[2134. 最少交换次数来组合所有的 1 II](https://leetcode.cn/problems/minimum-swaps-to-group-all-1s-together-ii/)

> **交换** 定义为选中一个数组中的两个 **互不相同** 的位置并交换二者的值。
>
> **环形** 数组是一个数组，可以认为 **第一个** 元素和 **最后一个** 元素 **相邻** 。
>
> 给你一个 **二进制环形** 数组 `nums` ，返回在 **任意位置** 将数组中的所有 `1` 聚集在一起需要的最少交换次数。

---

两次滑窗，求连续的 0 和连续的 1，这样我们可以得到所有的情况，缺点是在边界的时候会重复计算。

```go
func minSwaps(nums []int) int {
    zero, one := 0, 0
    for _, x := range nums {
        if x == 0 {
            zero ++
        } else {
            one ++
        }
    }
    if zero == 0 || zero == len(nums) {
        return 0
    }
    sum := 0
    ans := 0x3f3f3f3f
    for i, x := range nums {
        sum += x
        if i < zero - 1 {
            continue
        }
        ans = min(ans, sum)
        sum -= nums[i - zero + 1]
    }
    sum = 0
    for i, x := range nums {
        sum += x
        if i < one - 1 {
            continue
        }
        ans = min(ans, one - sum)
        sum -= nums[i - one + 1]
    }
    return ans
}
```

另一种方法，取模运算

```go
func minSwaps(nums []int) int {
    zero, one := 0, 0
    n := len(nums)
    for _, x := range nums {
        if x == 0 {
            zero ++
        } else {
            one ++
        }
    }
    if zero == 0 || zero == len(nums) {
        return 0
    }
    sum := 0
    ans := 0x3f3f3f3f
    for i := 0; i < n + zero; i ++ {
        sum += nums[i % n]
        if i < zero - 1 {
            continue
        }
        ans = min(ans, sum)
        sum -= nums[(i - zero + 1) % n]
    }
    return ans
}
```

