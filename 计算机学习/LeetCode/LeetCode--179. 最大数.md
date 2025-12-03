[179. 最大数](https://leetcode.cn/problems/largest-number/)

> 给定一组非负整数 `nums`，重新排列每个数的顺序（每个数不可拆分）使之组成一个最大的整数。
>
> **注意：**输出结果可能非常大，所以你需要返回一个字符串而不是整数。

---

直接sort.slice，这里的判定规则需要自定义

通过ans[i] + [j] < ans[j] + ans[i]，确保拼接后能得到最大的数字。

```go
func largestNumber(nums []int) string {
    var ans []string
    for _,v := range nums {
        ans = append(ans, strconv.Itoa(v))
    }
    sort.Slice(ans, func(i, j int) bool {
        return ans[i] + ans[j] > ans[j] + ans[i]
    })
    if ans[0] == "0" {
        return "0"
    }
    return strings.Join(ans, "")
}
```

