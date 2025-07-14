[658. 找到 K 个最接近的元素](https://leetcode.cn/problems/find-k-closest-elements/)

> 给定一个 **排序好** 的数组 `arr` ，两个整数 `k` 和 `x` ，从数组中找到最靠近 `x`（两数之差最小）的 `k` 个数。返回的结果必须要是按升序排好的。
>
> 整数 `a` 比整数 `b` 更接近 `x` 需要满足：
>
> - `|a - x| < |b - x|` 或者
> - `|a - x| == |b - x|` 且 `a < b`

---

二分找到，然后双指针移动指针

```go
func findClosestElements(arr []int, k int, x int) []int {
    idx := sort.SearchInts(arr, x)
    l, r := idx - 1, idx
    for r - l <= k {
        if l < 0 {
            r ++
        } else if r >= len(arr) || x - arr[l] <= arr[r] - x {
            l --
        } else {
            r ++
        }
    }
    
    return arr[l + 1 : r]
}
```

