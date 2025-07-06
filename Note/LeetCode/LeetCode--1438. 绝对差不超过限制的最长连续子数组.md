[1438. 绝对差不超过限制的最长连续子数组](https://leetcode.cn/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)

> 给你一个整数数组 `nums` ，和一个表示限制的整数 `limit`，请你返回最长连续子数组的长度，该子数组中的任意两个元素之间的绝对差必须小于或者等于 `limit` *。*
>
> 如果不存在满足条件的子数组，则返回 `0` 。

---

用两个队列存储最大值和最小值，如果遍历到的值够大，或者更够小，就弹出队尾元素，然后将遍历到的元素加入队尾，如果当前队列队首已经超出了当前滑动窗口的范围，就弹出队首，然后我们只需要记录两个队列队首之差即可

```go
func longestSubarray(nums []int, limit int) int {
    var minQ, maxQ []int
    l := 0
    ans := 0
    for i, x := range nums {
        for len(minQ) > 0 && x <= nums[minQ[len(minQ) - 1]] {
            minQ = minQ[:len(minQ) - 1]
        }
        minQ = append(minQ, i)

        for len(maxQ) > 0 && x >= nums[maxQ[len(maxQ) - 1]] {
            maxQ = maxQ[:len(maxQ) - 1]
        }
        maxQ = append(maxQ, i)

        for nums[maxQ[0]] - nums[minQ[0]] > limit {
            l ++
            if minQ[0] < l {
                minQ = minQ[1:]
            }
            if maxQ[0] < l {
                maxQ = maxQ[1:]
            }
        }
        ans = max(ans, i - l + 1)
    }
    return ans
}
```

