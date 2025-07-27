[84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)

---

## 正文

题目如下

> 给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。
>
> 求在该柱状图中，能够勾勒出来的矩形的最大面积。

这道题暴力很简单，但是时间复杂度是O(N^2)，在这里我们不予考虑，我在这里主要介绍一下单调栈的做法。

单调栈主要的思路就是，将遍历到的元素下标压入栈中，如果当前遍历的元素小于栈顶元素，就没有遵循单调的原则，需要先把栈中大于当前遍历到的数字的元素弹出，再把当前遍历的元素压入栈中，在这个过程中，我们还需要重新计算最大的矩形面积。

**如何计算？**

- 首先我们需要明确，我们的栈是存储的下标，通过下标的差值我们可以计算出宽度，而且栈中元素是保持着单调递增的趋势，所以我们每次弹出的下标对应的数值都可以作为我们的矩形的高，这样便可以计算出我们的矩形面积。

**为什么这样做能得出正确答案？**

- 首先，我们需要先了解一下暴力的做法，暴力是通过两个for循环遍历来实现的矩形的面积最大值计算，而单调栈是只在每次弹出元素的时候重新计算矩形面积，每个元素最多入栈，出栈一次，所以时间复杂度远小于暴力做法。
- 但我们仔细想一下就能够知道，暴力做法是有很多多余的计算步骤的，比如以[1,2,3,4,5,6,1]为例子，遍历1的时候，会把2，3，4，5，6，1遍历完，显然效率很低，而单调栈在弹出元素时，能够确定，以当前下标对应的矩形的高的最大面积是多少，想清楚这一点，这道题就迎刃而解了。

**下面是代码**

```go
func largestRectangleArea(heights []int) int {
    var st []int
    ans := 0
    heights = append(heights, -1)
    for i := 0; i < len(heights); i ++ {
        for len(st) != 0 && heights[i] < heights[st[len(st) - 1]] {
            idx := st[len(st) - 1]
            st = st[:len(st) - 1]
            var l int
            if len(st) == 0 {
                l = -1
            } else {
                l = st[len(st) - 1]
            }
            ans = Max(ans, (i - l - 1) * heights[idx])
        }
        st = append(st, i)
    }
    return ans
 }

 func Max(a int, b int) int {
    if a >= b {
        return a
    }
    return b
 }
```

---

二刷

```go
func largestRectangleArea(heights []int) int {
    heights = append(heights, -1)
    st := []int{-1}
    ans := 0
    for i, x := range heights {
        for len(st) > 1 && heights[st[len(st) - 1]] >= x {
            idx := st[len(st) - 1]
            st = st[:len(st) - 1]
            left := st[len(st) - 1]
            ans = max(ans, (i - left - 1) * heights[idx])
        }
        st = append(st, i)
    }
    return ans
}
```

