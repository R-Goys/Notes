[4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)

> 给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。
>
> 算法的时间复杂度应该为 `O(log (m+n))` 。

----

难想，是我无法想出来的二分，有点顶级的题目，开背！

主要思路是将数组分成左右两个部分进行计算，代码内嵌详细注释

```go
func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    if len(nums1) > len(nums2) {
        //这一步主要为了之后能够正确地进行二分
        return findMedianSortedArrays(nums2, nums1)
    }
    n, m := len(nums1), len(nums2)
    mid := (n + m + 1) / 2
    Left, Right := 0, n
    for Left <= Right {
        //i为二分查找的中点
        i := (Left + Right) / 2
        //j为nums2数组对应二分查找的下标
        j := mid - i
        //为什么是mid - i？
        //因为我们查找中位数，i,j左边部分和长度一定为mid！
        if i > 0 && nums1[i - 1] > nums2[j] {
            Right = i - 1
        } else if i < n && nums1[i] < nums2[j - 1] {
            Left = i + 1
            //以上为二分查找，缩小边界的代码
            //当i == n时，说明左边部分一定包含了nums1数组
            //此时，i - 1可能是我们要找的中心元素中的其中一个
            //在下一步会进行判断
        } else {
            LeftNum := 0
            if i == 0 {
                //数组左部分不含nums1
                LeftNum = nums2[j - 1]
            } else if j == 0 {
                //此时j = 0，i必然=n
                //我们可以想象，由于此时j = 0，所以nums2的不存在左部分，所以i - 1一定是左边部分的最大值
                //所以下标为i - 1的数，必然为中心元素的其中一个元素
                LeftNum = nums1[i - 1]
            } else {
                //其他情况,两部分均被包含，此时取最大值即可
                LeftNum = max(nums2[j - 1], nums1[i - 1])
            }
            if (n + m) % 2 == 1 {
                return float64(LeftNum)
            }
				//接下来判断右部分
            RightNum := 0
            if j == m {
                //说明nums2均被包含在左侧部分
                RightNum = nums1[i]
            } else if i == n {
                //nums1均被包含在右侧部分
                RightNum = nums2[j]
            } else {
                //都被包含了，取最小值即可
                RightNum = min(nums1[i], nums2[j])
            }
            return float64(LeftNum + RightNum) / 2.0
        }
    }
    //没找到
    return 0.0
}
```

