[209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

> 给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**
>
> 找出该数组中满足其总和大于等于 `target` 的长度最小的 **子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

---

时间O(N)，空间O(1)解法，滑动窗口思想，利用双指针维护sum。

```go
func minSubArrayLen(target int, nums []int) int {
    pre := 0
    ne := -1
    sum := 0
    MinCnt := 114514
    n := len(nums)
    for pre < n && ne < n {
        if sum < target {
            ne ++
            if ne < n {
                sum += nums[ne]
            }
        } else {
            MinCnt = min(MinCnt, ne - pre + 1)
            sum -= nums[pre]
            pre ++
        }
    }
    if MinCnt == 114514 {
        return 0
    }
    return MinCnt
}
```

时间复杂度O(NlogN)，空间O(N)，前缀和+滑动窗口，先计算前缀和，第一层for循环，使用i表示我们的前缀起始的边界，利用他，通过变换target和前缀和经过一定计算变成我们的二分查找的目标值，使得可以找到最近的前缀和，使得该前缀和减去当前for循环起始边界的前缀和可以得到总和大于等于target的子数组，随后用下标相减就是我们的子数组长度。

```go
func minSubArrayLen(target int, nums []int) int {
    n := len(nums)

    ans := 114514
    pre := make([]int, n + 1)

    for i := 1; i <= n; i ++ {
        pre[i] = pre[i - 1] + nums[i - 1]
    }
    for i := 1; i <= n; i ++ {
        aim := target + pre[i - 1]
        offset := Search(pre, aim)
        if offset >= 0 && offset <= n {
            ans = min(ans, offset - (i - 1))
        }
    }
    if ans == 114514 {
        return 0
    }
    return ans
}

func Search(nums []int, target int) int {
    l, r := 0, len(nums)
	for l < r {
		mid := (l + r) / 2
		if nums[mid] >= target {
			r = mid
		} else {
			l = mid + 1
		}
	}
    return r
}
```

