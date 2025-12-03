[75. 颜色分类](https://leetcode.cn/problems/sort-colors/)

>给定一个包含红色、白色和蓝色、共 `n` 个元素的数组 `nums` ，**[原地](https://baike.baidu.com/item/原地算法)** 对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。
>
>我们使用整数 `0`、 `1` 和 `2` 分别表示红色、白色和蓝色。
>
>必须在不使用库内置的 sort 函数的情况下解决这个问题。

----

三指针：

```go
func sortColors(nums []int)  {
    red := 0
    white := 0
    blue := 0
    for i := 0; i < len(nums); i ++ {
        if nums[i] == 0 {
            nums[blue] = 2
            blue++
            nums[white] = 1
            white++
            nums[red] = 0
            red++
        } else if nums[i] == 1 {
            nums[blue] = 2
            blue++
            nums[white] = 1
            white++
        } else {
            nums[blue] = 2
            blue++
        }
    }
}
```

二刷：

说一下思路，三个指针是滚动前进的一旦前面的该颜色前面的颜色被更新了，那么就应当把当前这个元素推到下一个位置，因为这个颜色在最前面的元素必然会被覆盖
