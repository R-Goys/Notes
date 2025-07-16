[1283. 使结果不超过阈值的最小除数](https://leetcode.cn/problems/find-the-smallest-divisor-given-a-threshold/)

> 给你一个整数数组 `nums` 和一个正整数 `threshold` ，你需要选择一个正整数作为除数，然后将数组里每个数都除以它，并对除法结果求和。
>
> 请你找出能够使上述结果小于等于阈值 `threshold` 的除数中 **最小** 的那个。
>
> 每个数除以除数后都向上取整，比方说 7/3 = 3 ， 10/2 = 5 。
>
> 题目保证一定有解。

---

做法有点懒加载的思想，直接二分，当我们需要对某个下标索引进行验证的时候才进行求和计算

```go
func smallestDivisor(nums []int, threshold int) int {
    maxn := -1
    for _, x := range nums {
        maxn = max(x, maxn)
    }
    sort.Slice(nums, func(i, j int) bool {
        return nums[i] < nums[j]
    })
    div := sort.Search(maxn - 1, func(i int) bool {
        return sum(nums, i) <= threshold
    })
    return div + 1
}

func sum(nums []int, div int) int {
    div += 1
    ans := 0
    for _, x := range nums {
        if x % div != 0 {
            ans += x / div + 1
        } else {
            ans += x / div
        }
    }
    return ans
}
```

