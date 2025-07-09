[1385. 两个数组间的距离值](https://leetcode.cn/problems/find-the-distance-value-between-two-arrays/)

> 给你两个整数数组 `arr1` ， `arr2` 和一个整数 `d` ，请你返回两个数组之间的 **距离值** 。
>
> 「**距离值**」 定义为符合此距离要求的元素数目：对于元素 `arr1[i]` ，不存在任何元素 `arr2[j]` 满足 `|arr1[i]-arr2[j]| <= d` 。

---

对 arr2 排序，然后二分查找对应的 [x - d, x + d] 区间是否存在数字

```go
func findTheDistanceValue(arr1 []int, arr2 []int, d int) int {
    sort.Ints(arr2)
    ans := 0
    for _, x := range arr1 {
        i := bio(arr2, x - d)
        if i == len(arr2) || arr2[i] > x + d {
            ans++
        }
    }
    return ans
}

func bio(nums []int, target int) int {
    l, r := 0, len(nums)
    for l < r {
        mid := (l + r) / 2
        if nums[mid] < target {
            l = mid + 1
        } else {
            r = mid
        }
    }
    return l
}
```

