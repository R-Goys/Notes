[744. 寻找比目标字母大的最小字母](https://leetcode.cn/problems/find-smallest-letter-greater-than-target/)

> 给你一个字符数组 `letters`，该数组按**非递减顺序**排序，以及一个字符 `target`。`letters` 里**至少有两个不同**的字符。
>
> 返回 `letters` 中大于 `target` 的最小的字符。如果不存在这样的字符，则返回 `letters` 的第一个字符。

---

灵神的二分讲解真的神了，比目标字母大，即搜 `target + 1` 就行。

```go
func nextGreatestLetter(letters []byte, target byte) byte {
    idx := bio(letters, target + 1)
    if letters[idx] <= target {
        idx = 0
    }
    return letters[idx]
}

func bio(letters []byte, target byte) int {
    l, r := 0, len(letters) - 1
    for l < r {
        mid := (l + r) / 2
        if letters[mid] < target {
            l = mid + 1
        } else {
            r = mid
        }
    }
    return l
}
```

