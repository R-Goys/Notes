[88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

> 给你两个按 **非递减顺序** 排列的整数数组 `nums1` 和 `nums2`，另有两个整数 `m` 和 `n` ，分别表示 `nums1` 和 `nums2` 中的元素数目。
>
> 请你 **合并** `nums2` 到 `nums1` 中，使合并后的数组同样按 **非递减顺序** 排列。
>
> **注意：**最终，合并后数组不应由函数返回，而是存储在数组 `nums1` 中。为了应对这种情况，`nums1` 的初始长度为 `m + n`，其中前 `m` 个元素表示应合并的元素，后 `n` 个元素为 `0` ，应忽略。`nums2` 的长度为 `n` 。

---

### 暴力

第一次瞎写的

```go
func merge(nums1 []int, m int, nums2 []int, n int)  {
    for i := 0; i < n; i ++ {
        l, r := 0, m - 1
        for l <= r {
            mid := (l + r) / 2
            if nums1[mid] > nums2[i] {
                r = mid - 1
            } else {
                l = mid + 1
            }
        }
        copy(nums1[l + 1:], nums1[l:])
        nums1[l] = nums2[i]
        m++
    }
}
```

### 双指针

整体思路是定义指针，分别指向数组A和数组B的最后一个有效元素，从后向前填充数组A，易知(我第一次没想到)，后面填充的不会影响之前的元素，我们可能的想法是，万一B的元素填的太多了，覆盖掉A怎么办？事实上，这是不会影响的。

```go
func merge(nums1 []int, m int, nums2 []int, n int)  {
    i, j, k := m - 1, n - 1, m + n - 1
    for j >= 0 {
        if i >= 0 && nums1[i] > nums2[j] {
            nums1[k] = nums1[i]
            i --
        } else if j >= 0 {
            nums1[k] = nums2[j]
            j --
        }
        k --
    }
}
```

