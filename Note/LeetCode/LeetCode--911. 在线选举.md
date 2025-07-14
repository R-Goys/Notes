[911. 在线选举](https://leetcode.cn/problems/online-election/)

> 给你两个整数数组 `persons` 和 `times` 。在选举中，第 `i` 张票是在时刻为 `times[i]` 时投给候选人 `persons[i]` 的。
>
> 对于发生在时刻 `t` 的每个查询，需要找出在 `t` 时刻在选举中领先的候选人的编号。
>
> 在 `t` 时刻投出的选票也将被计入我们的查询之中。在平局的情况下，最近获得投票的候选人将会获胜。
>
> 实现 `TopVotedCandidate` 类：
>
> - `TopVotedCandidate(int[] persons, int[] times)` 使用 `persons` 和 `times` 数组初始化对象。
> - `int q(int t)` 根据前面描述的规则，返回在时刻 `t` 在选举中领先的候选人的编号。

---

抓住 times 数组是有序这个特点，我们可以通过 times 二分来确定下标，然后直接可以检索到对应时间的对应的领先候选人编号，这里我们直接用一个数组来存储即可，只需要保证同一下标对应的元素准确即可。

```go
type TopVotedCandidate struct {
    tops []int
    times []int
}


func Constructor(persons []int, times []int) TopVotedCandidate {
    tops := make([]int, 0)
    top := -1
    cnt := map[int]int{}
    cnt[-1] = -1
    for _, p := range persons {
        cnt[p] ++
        if cnt[top] <= cnt[p] {
            top = p
        }
        tops = append(tops, top)
    }
    return TopVotedCandidate{
        times: times,
        tops: tops,
    }
}


func (this *TopVotedCandidate) Q(time int) int {
    return this.tops[sort.SearchInts(this.times, time + 1) - 1]
}
```

