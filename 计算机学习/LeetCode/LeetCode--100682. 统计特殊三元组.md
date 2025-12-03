[100682. 统计特殊三元组](https://leetcode.cn/problems/count-special-triplets/)

> 给你一个整数数组 `nums`。
>
> **特殊三元组** 定义为满足以下条件的下标三元组 `(i, j, k)`：
>
> - `0 <= i < j < k < n`，其中 `n = nums.length`
> - `nums[i] == nums[j] * 2`
> - `nums[k] == nums[j] * 2`
>
> 返回数组中 **特殊三元组** 的总数。
>
> 由于答案可能非常大，请返回结果对 `109 + 7` 取余数后的值。

---

爽麻了，`O(N^2)` 必定会超时，想了半天 `O(NlogN)` 的方法，超时了无数次

```go
func specialTriplets(nums []int) int {
    mp := make(map[int][]int)
    ans := 0
    mod := 1000000007
    for k, x := range nums {
        mp[x] = append(mp[x], k)
    }
    for j, x := range nums {
        if len(mp[x * 2]) != 0 {
            left := sort.SearchInts(mp[x * 2], j)
            right := sort.SearchInts(mp[x * 2], j + 1)
            in1 := left
            in2 := len(mp[x * 2]) - right
            ans = (ans + in1 * in2) % mod
        }
    }
    return ans % mod
}

```

