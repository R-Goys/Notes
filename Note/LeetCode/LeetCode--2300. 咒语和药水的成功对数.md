[2300. 咒语和药水的成功对数](https://leetcode.cn/problems/successful-pairs-of-spells-and-potions/)

> 给你两个正整数数组 `spells` 和 `potions` ，长度分别为 `n` 和 `m` ，其中 `spells[i]` 表示第 `i` 个咒语的能量强度，`potions[j]` 表示第 `j` 瓶药水的能量强度。
>
> 同时给你一个整数 `success` 。一个咒语和药水的能量强度 **相乘** 如果 **大于等于** `success` ，那么它们视为一对 **成功** 的组合。
>
> 请你返回一个长度为 `n` 的整数数组 `pairs`，其中 `pairs[i]` 是能跟第 `i` 个咒语成功组合的 **药水** 数目。

---

对 potions 排序，然后二分查找对应的 `target` 的索引，然后直接加入 ans 数组即可。

难点是如何我们如何计算 `target`，这里的 +1，-1 是为了处理一些边界问题，虽然我也不是很明白，记住就行。

```go
func successfulPairs(spells []int, potions []int, success int64) []int {
    sort.Ints(potions)
    ans := []int{}
    for _, x := range spells {
        target := int(success - 1) / x
        idx := bio(potions, target + 1)
        ans = append(ans, len(potions) - idx)
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

