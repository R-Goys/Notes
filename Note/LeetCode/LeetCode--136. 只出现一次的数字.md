[136. 只出现一次的数字](https://leetcode.cn/problems/single-number/)

## 位运算

记得就会做，不记得就不会

```go
func singleNumber(nums []int) int {
    single := 0
    for _, num := range nums {
        single ^= num
    }
    return single
}
```

二刷复习

```go
func singleNumber(nums []int) int {
    n := len(nums)
    ans := 0
    for i := 0; i < n; i ++ {
        ans ^= nums[i]
    }
    return ans
}
```

