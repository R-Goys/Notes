[347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)

> 给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。

---

手撕堆排，说实话写着是真累哈。

```go
func topKFrequent(nums []int, k int) []int {
    var heap [][2]int
    mp := map[int]int{}
    for _, num := range nums {
        mp[num] ++
    }
    for k, v := range mp {
        push(&heap, [2]int{k, v})
    }
    var ans []int
    for i := 0; i < k; i ++ {
        ans = append(ans, pop(&heap))
    }
    return ans
}

func push(heap *[][2]int, num [2]int) {
    (*heap) = append((*heap), num)
    up(heap, len(*heap) - 1)
}

func up(heap *[][2]int, pos int) {
    u := pos
    if (*heap)[u][1] > (*heap)[pos / 2][1] {
        u = pos / 2
    }

    if u != pos {
        (*heap)[pos], (*heap)[u] = (*heap)[u], (*heap)[pos]
        up(heap, u)
    }
}

func pop(heap *[][2]int) int {
    ret := (*heap)[0][0]
    (*heap)[0], (*heap)[len(*heap) - 1] = (*heap)[len(*heap) - 1], (*heap)[0]
    (*heap) = (*heap)[:len(*heap) - 1]
    Down(heap, 0)
    return ret
}

func Down(heap *[][2]int, pos int) {
    u := pos
    if pos * 2 < len(*heap) && (*heap)[u][1] < (*heap)[pos * 2][1] {
        u = pos * 2
    }
    if pos * 2 + 1 < len(*heap) && (*heap)[u][1] < (*heap)[pos * 2 + 1][1] {
        u = pos * 2 + 1
    }
    if u != pos {
        (*heap)[u], (*heap)[pos] = (*heap)[pos], (*heap)[u]
        Down(heap, u)
    }
}
```

