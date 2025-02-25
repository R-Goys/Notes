[31. 下一个排列](https://leetcode.cn/problems/next-permutation/)

>整数数组的一个 **排列** 就是将其所有成员以序列或线性顺序排列。
>
>- 例如，`arr = [1,2,3]` ，以下这些都可以视作 `arr` 的排列：`[1,2,3]`、`[1,3,2]`、`[3,1,2]`、`[2,3,1]` 。
>
>整数数组的 **下一个排列** 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 **下一个排列** 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。
>
>- 例如，`arr = [1,2,3]` 的下一个排列是 `[1,3,2]` 。
>- 类似地，`arr = [2,3,1]` 的下一个排列是 `[3,1,2]` 。
>- 而 `arr = [3,2,1]` 的下一个排列是 `[1,2,3]` ，因为 `[3,2,1]` 不存在一个字典序更大的排列。
>
>给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。
>
>必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。

---

### 第一次做

贵物题目，虽然想出来了思路是怎么做的，但是代码写不出来💀,无所谓了,开背

```go
func nextPermutation(nums []int)  {
    if len(nums) <= 1 {
        return
    }
    
    i, j, k := len(nums) - 2, len(nums) - 1, len(nums) - 1

    for i >= 0 && nums[i] >= nums[j] {
        i --
        j --
    }
    
    if i >= 0 {
		for nums[i] >= nums[k] {
			k--
		}
		nums[i], nums[k] = nums[k], nums[i]
    }


	for i, j := j, len(nums)-1; i < j; i, j = i+1, j-1 {
		nums[i], nums[j] = nums[j], nums[i]
	}
}
```

### 二刷

有思路，写出来了，但是40分钟...

首先从后往前遍历，找到第一个前一个元素`i`小于后一个元素`j`的数字，若能够找到，则证明找到的下标之后的排列均为升序，只需要在这里面找到第一个大于`i`的元素，然后交换位置，最后将`nums[j:]`这部分反转一下就是我们最后的答案。

如果没有找到这样的两个元素，说明当前数组是完全的降序排列，相当于是最后一个排列，直接反转数组即可。

```go
func nextPermutation(nums []int)  {
    if len(nums) <= 1 {
        return
    }

    i, j := len(nums) - 2, len(nums) - 1
    for i >= 0 && nums[i] >= nums [j] {
        i --
        j --
    }
    firstBig := len(nums) - 1
    if i >= 0 {
        for nums[i] >= nums[firstBig] {
            firstBig --
        }
        nums[firstBig], nums[i] = nums[i], nums[firstBig]
    }
    for l, r := j, len(nums) - 1; l < r; {
        nums[l],nums[r] = nums[r], nums[l]
        l ++
        r --
    }
}
```



