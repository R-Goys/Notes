[1170. 比较字符串最小字母出现频次](https://leetcode.cn/problems/compare-strings-by-frequency-of-the-smallest-character/)

> 定义一个函数 `f(s)`，统计 `s` 中**（按字典序比较）最小字母的出现频次** ，其中 `s` 是一个非空字符串。
>
> 例如，若 `s = "dcce"`，那么 `f(s) = 2`，因为字典序最小字母是 `"c"`，它出现了 2 次。
>
> 现在，给你两个字符串数组待查表 `queries` 和词汇表 `words` 。对于每次查询 `queries[i]` ，需统计 `words` 中满足 `f(queries[i])` < `f(W)` 的 **词的数目** ，`W` 表示词汇表 `words` 中的每个词。
>
> 请你返回一个整数数组 `answer` 作为答案，其中每个 `answer[i]` 是第 `i` 次查询的结果。

---

先统计 f(words...)， 排序后二分查找。

```go
func numSmallerByFrequency(queries []string, words []string) []int {
    src := make([]int, len(words))
    ans := make([]int, 0)
    for _, x := range words {
        mp := ['z' + 1]int{}
        minn := 'z' + 1
        for _, b := range x {
            mp[b] ++
            minn = min(b, minn)
        }
        src = append(src, mp[minn])
    }
    sort.Ints(src)

    for _, x := range queries {
        mp := ['z' + 1]int{}
        minn := 'z' + 1
        for _, b := range x {
            mp[b] ++
            minn = min(b, minn)
        }
        ans = append(ans, len(src) - sort.SearchInts(src, mp[minn] + 1))
    }
    return ans
}
```

