[1191. K 次串联后最大子数组之和](https://leetcode.cn/problems/k-concatenation-maximum-sum/)

> 给定一个整数数组 `arr` 和一个整数 `k` ，通过重复 `k` 次来修改数组。
>
> 例如，如果 `arr = [1, 2]` ， `k = 3` ，那么修改后的数组将是 `[1, 2, 1, 2, 1, 2]` 。
>
> 返回修改后的数组中的最大的子数组之和。注意，子数组长度可以是 `0`，在这种情况下它的总和也是 `0`。
>
> 由于 **结果可能会很大**，需要返回的 `109 + 7` 的 **模** 。

---

幻想一下，我们最终的答案有三种可能，设原数组 arr 可以表示为 [x, y, z,...]：

1. 一种情况的答案为： [x，**y，z，...，x，y，...，m，...**] 加粗为被选中的子数组，此时最大，其实就是将两个数组拼接起来，取最大的部分。
2. 另一种情况就是当我们的数组总和大于 0 ，这个意思也就是说，我们除了拼接两个数组取中间部分，此时 k > 2的话，我们可以将所有的数组都插入到这两个数组的中间，因为他们的贡献是一定大于 0 的。
3. 最后一种，就是 k == 1 ，此时就直接使用原本的求最大子数组和的方法即可。

```c
var mod = 1000000007

func kConcatenationMaxSum(arr []int, k int) int {
    sum := 0
    n := len(arr)
    for i := 0; i < n; i ++ {
        sum += arr[i]
    }
    if k == 1 {
        return MaxSum(arr)
    } else if sum > 0 {
        return MaxSum(append(arr, arr...)) + sum * (k - 2) % mod
    }
    return MaxSum(append(arr, arr...))
}

func MaxSum(nums []int) int {
    n := len(nums)
    pre, MaxVal := 0, 0
    for i := 0; i < n; i ++ {
        pre = max(pre + nums[i], nums[i])
        MaxVal = max(MaxVal, pre)
    }
    return MaxVal % mod
}
```

