[16. 最接近的三数之和](https://leetcode.cn/problems/3sum-closest/)

> 给你一个长度为 `n` 的整数数组 `nums` 和 一个目标值 `target`。请你从 `nums` 中选出三个整数，使它们的和与 `target` 最接近。
>
> 返回这三个数的和。
>
> 假定每组输入只存在恰好一个解。

---

相比于三数之和，多了一点，就在于维护一个变量`distance`来负责实时更新我们的最接近的值。

```go
func threeSumClosest(nums []int, target int) int {
        sort.Ints(nums)
        distance, ans := 20001, -1
        length := len(nums)
        for i := 0; i < length; i ++ {
                prePtr, proPtr := i + 1, length - 1
                for prePtr < proPtr {
                        if distance > abs(target - nums[prePtr] - nums[proPtr] - nums[i]) {
                                distance = abs(nums[prePtr] + nums[proPtr] + nums[i] - target)
                                ans = nums[i] + nums[prePtr] + nums[proPtr]
                        }
                        if nums[prePtr] + nums[proPtr] + nums[i] < target {
                                prePtr ++
                        } else {
                                proPtr --
                        }
                }
        } 
        return ans
}

func abs(num int) int {
        if num > 0 {
                return num
        }
        return -num
}
```

试了一下两个缩进的代码，感觉挺新鲜的..