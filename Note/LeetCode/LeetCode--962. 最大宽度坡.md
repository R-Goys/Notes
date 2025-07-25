[962. 最大宽度坡](https://leetcode.cn/problems/maximum-width-ramp/)

> 给定一个整数数组 `A`，*坡*是元组 `(i, j)`，其中 `i < j` 且 `A[i] <= A[j]`。这样的坡的宽度为 `j - i`。
>
> 找出 `A` 中的坡的最大宽度，如果不存在，返回 0 。

---

依旧单调栈，栈中的数字单调递减，之后更大的数字没有必要加入栈中，然后每次遇到数字时，在栈中进行二分查找即可。

```go
func maxWidthRamp(nums []int) int {
    st := make([]int, 0)
    ans := 0
    for r, x := range nums {
        if len(st) != 0 {
            l := sort.Search(len(st) - 1, func(i int) bool {
                return nums[st[i]] <= x
            })
            if nums[st[l]] <= x {
                ans = max(ans, r - st[l])
            }
        }
        if len(st) == 0 || nums[st[len(st) - 1]] > x {
            st = append(st, r)
        }
    }
    return ans
}
```

