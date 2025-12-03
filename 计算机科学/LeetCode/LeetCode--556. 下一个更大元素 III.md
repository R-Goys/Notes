[556. 下一个更大元素 III](https://leetcode.cn/problems/next-greater-element-iii/)

> 给你一个正整数 `n` ，请你找出符合条件的最小整数，其由重新排列 `n` 中存在的每位数字组成，并且其值大于 `n` 。如果不存在这样的正整数，则返回 `-1` 。
>
> **注意** ，返回的整数应当是一个 **32 位整数** ，如果存在满足题意的答案，但不是 **32 位整数** ，同样返回 `-1` 。

---

和下一个排列这道题类似，只需要将该题的int变成字节切片就可以了。

首先是**从后往前**遍历，找到第一个前一个数字小于后一个数字的位置，如果没找到，就说明当前数字是单调递减的序列，没有下一个排列，此时i<0返回-1，如果不是，则进行下一步。

我们现在已经找到了从后往前第一个发生"错位"的地方，此时，如果再去根据这个数字从后往前找到第一个大于他的数，将他们交换，这样，我们的第i位就已经确定是当前排列的正确的数字，但是此时[j:]的部分还是降序排列的，我们此时的更高的位已经变得更大，所以我们的之后的位应该变成最小值，所以此时直接倒置即可。

```go
func nextGreaterElement(n int) int {
    num := strconv.Itoa(n)
    NumBytes := []byte(num)
    n = len(NumBytes)
    if n <= 1 {
        return -1
    }
    i, j := n - 2, n - 1
    for i >= 0 && NumBytes[i] >= NumBytes[j] {
        i --
        j --
    }
    if i < 0 {
        return -1
    }
    firstBig := n - 1
    if i >= 0 {
        for NumBytes[i] >= NumBytes[firstBig] {
            firstBig --
        }
        NumBytes[firstBig], NumBytes[i] = NumBytes[i], NumBytes[firstBig]
    }
    for l, r := j, n - 1; l < r; {
        NumBytes[l],NumBytes[r] = NumBytes[r], NumBytes[l]
        l ++
        r --
    }
    ans, _ := strconv.Atoi(string(NumBytes))
    if ans > math.MaxInt32 {
        return -1
    }
    return ans
}
```

值得注意的是题目描述，如果超出int32的范围，需要返回-1