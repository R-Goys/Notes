[15. 三数之和](https://leetcode.cn/problems/3sum/)

>给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。
>
>**注意：**答案中不可以包含重复的三元组。

----

### 正文

三数之和，总体思路是排序+双指针，相较于暴力的O(n^3)优化了不少，遍历过程中也可以适当的剪枝来进一步优化,

具体思路如下：

1. 先排序，将数组变成有序的，数组长度为n
2. 随后第一层遍历元素i
3. 定义双指针:j,k以及目标值target,其中，j的初始值为i + 1, k的初始值为n - 1, target为-nums[i].
4. 循环遍历j，循环内嵌k，当nums[k] + nums[j] > target时，k--，最后如果找不到相应的target，则进行continue，若找到了，则加入答案中。
5. 在遍历过程中，如果nums[i] > 0时，可以直接返回答案。

具体代码如下：

```go
func threeSum(nums []int) [][]int {
    n := len(nums)
    sort.Ints(nums)
    ans := make([][]int, 0)
    for i := 0; i < n; i ++ {
        if nums[i] > 0 {
            return ans
        }
        if i > 0 && nums[i] == nums[i - 1] {
            continue
        }

        k := n - 1
        target := -nums[i]

        for j := i + 1; j < n; j ++ {
            if j > i + 1 && nums[j] == nums[j - 1] {
                continue
            }
            for j < k && nums[j] + nums[k] > target {
                k --
            }

            if j == k {
                break
            } else if target == nums[k] + nums[j] {
                ans = append(ans, []int{nums[i], nums[j], nums[k]})
            }
        }
    }
    return ans
}
```

----

