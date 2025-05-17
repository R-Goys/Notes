[1981. 最小化目标值与所选元素的差](https://leetcode.cn/problems/minimize-the-difference-between-target-and-chosen-elements/)

> 给你一个大小为 `m x n` 的整数矩阵 `mat` 和一个整数 `target` 。
>
> 从矩阵的 **每一行** 中选择一个整数，你的目标是 **最小化** 所有选中元素之 **和** 与目标值 `target` 的 **绝对差** 。
>
> 返回 **最小的绝对差** 。
>
> `a` 和 `b` 两数字的 **绝对差** 是 `a - b` 的绝对值。

---

分组背包问题，感觉分组背包这部分题目写得比较少，不是很熟悉，需要多练练。

分组背包需要先遍历组，再从大到小遍历体积，然后才是遍历组中元素，除了这个之外，我们这道题还有一个重点，就是如何确定背包的容量大小，也就是状态转移需要开辟的空间，我们可以分析一下，如果我们每一行选择最小的值，这样也比 target 更大，那么最优只能是选择最小的，除此之外，我们就不会得到比 target 更大的差值。



```go
func minimizeTheDifference(mat [][]int, target int) int {
    dp := make([]bool, min(len(mat) * 70, target * 2) + 1)
    dp[0] = true
    minSum, maxSum := 0, 0
    for _, row := range mat {
        mi, mx := row[0], row[0]
        for _, v := range row[1:] {
            if v > mx {
                mx = v
            } else if v < mi {
                mi = v
            }
        }
        // 累加当前行最小值，便于之后计算
        minSum += mi
        // 获取前面所有行可以到达得最大值，且不超过 target * 2
        maxSum = min(maxSum + mx, target * 2)
        // 分组背包
        for j := maxSum; j >= 0; j -- {
            dp[j] = false
            for _, v := range row {
                if v <= j && dp[j - v] {
                    dp[j] = true
                    break
                }
            }
        }
    }
    // 获取 ans，通过之前的所有行最小值以及其他所有的有效体积进行计算。
    ans := abs(minSum - target)
    for i, ok := range dp {
        if ok {
            ans = min(ans, abs(i - target))
        }
    }
    return ans
}

func abs(x int) int {
    if x > 0 {
        return x
    }
    return -x
}
```

