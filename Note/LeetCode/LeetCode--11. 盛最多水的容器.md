[11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

> 给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。
>
> 找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。
>
> 返回容器可以储存的最大水量。
>
> **说明：**你不能倾斜容器。

经典双指针，关于这个写法，有一句解释我觉得是最好的。

> **一句话概括：我们left++和right--都是为了尝试取到更多的水，如果短的板不动的话，取到的水永远不会比上次多。**

```go
func maxArea(height []int) int {
    left, right := 0, len(height) - 1
    ans := 0
    for left < right {
        ans = max(ans, min(height[right], height[left]) * (right - left))
        if height[left] > height[right] {
            right --
        } else {
            left ++
        }
    }
    return ans
}
```

