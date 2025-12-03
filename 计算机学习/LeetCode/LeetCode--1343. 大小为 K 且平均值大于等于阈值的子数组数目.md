[1343. 大小为 K 且平均值大于等于阈值的子数组数目](https://leetcode.cn/problems/number-of-sub-arrays-of-size-k-and-average-greater-than-or-equal-to-threshold/)

> 给你一个整数数组 `arr` 和两个整数 `k` 和 `threshold` 。
>
> 请你返回长度为 `k` 且平均值大于等于 `threshold` 的子数组数目。

---

虽然很简单，但是适合很久没写滑动窗口的人来练练手。

```go
func numOfSubarrays(arr []int, k int, threshold int) int {
    n := len(arr)
    l, r := 0, -1
    sum := 0
    ans := 0
    for r < n - 1 {
        for r - l + 1 < k && r < n - 1 {
            r ++
            sum += arr[r]
        }
        if r - l + 1 == k && l < n {
            if threshold <= sum / k {
                ans ++
            }
            sum -= arr[l]
            l ++
        }
    }
    return ans
}
```

另外一种方法，参考灵神，无需维护双指针，直接遍历，这也是一个有点意思的方法：

```go
func numOfSubarrays(arr []int, k int, threshold int) int {
    sum := 0
    ans := 0
    for i, x := range arr {
        sum += x
        if i < k - 1 {
            continue
        }
        if sum >= k * threshold {
            ans ++
        }
        sum -= arr[i - k + 1]
    }
    return ans
}
```

