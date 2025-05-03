[918. 环形子数组的最大和](https://leetcode.cn/problems/maximum-sum-circular-subarray/)

> 给定一个长度为 `n` 的**环形整数数组** `nums` ，返回 *`nums` 的非空 **子数组** 的最大可能和* 。
>
> **环形数组** 意味着数组的末端将会与开头相连呈环状。形式上， `nums[i]` 的下一个元素是 `nums[(i + 1) % n]` ， `nums[i]` 的前一个元素是 `nums[(i - 1 + n) % n]` 。
>
> **子数组** 最多只能包含固定缓冲区 `nums` 中的每个元素一次。形式上，对于子数组 `nums[i], nums[i + 1], ..., nums[j]` ，不存在 `i <= k1, k2 <= j` 其中 `k1 % n == k2 % n` 。

---

如果将拼接两个数组展开的话，选择的环形子数组有两种情况：

[begin，sec，**thr，...，lastthr，lastsec，lastone，begin**，sec，thr，...]

或者

[begin，**sec，thr，...，lastthr**，lastsec，lastone，begin，...]

区别是，在向后重复一个数组的基础上，跨越了一个原数组的最后一个元素，而另一种情况则仅仅是对原数组进行最大子数组计算，它的结果就是环形最大子数组的结果，我们可以发现，一种直接就是对原数组进行计算，而另一种，我们可以看作是对原数组的最小值取余：

```go
func maxSubarraySumCircular(nums []int) int {
    n := len(nums)
    preMax, maxRes := nums[0], nums[0]
    preMin, minRes := nums[0], nums[0]
    sum := nums[0]
    for i := 1; i < n; i++ {
        preMax = max(preMax + nums[i], nums[i])
        maxRes = max(maxRes, preMax)
        preMin = min(preMin + nums[i], nums[i])
        minRes = min(minRes, preMin)
        sum += nums[i]
    }
    if maxRes < 0 {
        return maxRes
    } else {
        return max(maxRes, sum - minRes)
    }
}
```



