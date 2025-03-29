[283. 移动零](https://leetcode.cn/problems/move-zeroes/)

> 给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。
>
> **请注意** ，必须在不复制数组的情况下原地对数组进行操作。

---

双指针，我的思路类似于红蓝白染色那道题，但是官方有一个通过交换后面元素的非0元素来实现，我觉得那个方法也很不错，都差不多。
```go
func moveZeroes(nums []int)  {
        zeroPtr, numPtr := 0, 0
        for numPtr < len(nums) {
                nums[zeroPtr] = nums[numPtr]
                if nums[zeroPtr] != 0 {
                        zeroPtr ++
                }
                numPtr ++
        }
        for zeroPtr < len(nums) {
                nums[zeroPtr] = 0
                zeroPtr ++
        }
        return
}
```

谁是0？你是0吗？