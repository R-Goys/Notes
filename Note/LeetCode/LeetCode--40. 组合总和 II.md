[40. 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)

> 给定一个候选人编号的集合 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。
>
> `candidates` 中的每个数字在每个组合中只能使用 **一次** 。
>
> **注意：**解集不能包含重复的组合。 

太恶心了，这个组合总和2，搞个去重这么几把麻烦。

虽然是看着答案才写出来的，不过也学到了一些新的思路

这道题就是将candidate中的元素及其出现的个数以键值对的形式存入数组的[0]和[1]，然后我们在做dfs的时候，只需要从每一种元素(种类相同代表数字相同)，的第一个元素开始找，这样，就不会出现从第二个元素开始找的情况，在这其中也可以适当剪枝，由于数组是有序的，所以一旦发现接下来要遍历到元素加起来如果大于target，那么可以直接返回，同时，由于切片底层共享数组，所以这里需要采取`copy`函数的形式进行拷贝。

```go
func combinationSum2(candidates []int, target int) [][]int {
    cur := make([]int, 0)
    ans := make([][]int, 0)
    
    sort.Ints(candidates)
    var freq [][2]int
    for _, num := range candidates {
        if freq == nil || num != freq[len(freq)-1][0] {
            freq = append(freq, [2]int{num, 1})
        } else {
            freq[len(freq)-1][1]++
        }
    }
    dfs(cur, target, 0, target, &ans, freq)
    return ans
}

func dfs(cur []int, target, pos, rest int, ans *[][]int, freq [][2]int) {
    if rest == 0 {
        tmp := make([]int, len(cur))
        copy(tmp, cur)
        (*ans) = append(*ans, tmp)
        return
    }
    if pos == len(freq) || rest < freq[pos][0] {
        return
    }

    dfs(cur, target, pos + 1, rest, ans, freq)

    most := min(rest/freq[pos][0], freq[pos][1])
    for i := 1; i <= most; i ++ {
        cur = append(cur, freq[pos][0])
        dfs(cur, target, pos + 1, rest - i * freq[pos][0], ans, freq)
    }
}
```

