[1287. 有序数组中出现次数超过25%的元素](https://leetcode.cn/problems/element-appearing-more-than-25-in-sorted-array/)

> 给你一个非递减的 **有序** 整数数组，已知这个数组中恰好有一个整数，它的出现次数超过数组元素总数的 25%。
>
> 请你找到并返回这个整数

---

二分，左右边界的查找，但是这个 span 的思想还有点意思，不好想出来。

```go
func findSpecialInteger(arr []int) int {
    limit := len(arr) / 4
    span := max(1, len(arr) / 4)
    for idx := 0; idx < len(arr); idx += span {
        l := sort.SearchInts(arr, arr[idx])
        r := sort.SearchInts(arr, arr[idx] + 1)
        if r - l > limit {
            return arr[l]
        }
    }
    return -1
}
```