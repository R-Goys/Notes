[1019. 链表中的下一个更大节点](https://leetcode.cn/problems/next-greater-node-in-linked-list/)

>给定一个长度为 `n` 的链表 `head`
>
>对于列表中的每个节点，查找下一个 **更大节点** 的值。也就是说，对于每个节点，找到它旁边的第一个节点的值，这个节点的值 **严格大于** 它的值。
>
>返回一个整数数组 `answer` ，其中 `answer[i]` 是第 `i` 个节点( **从1开始** )的下一个更大的节点的值。如果第 `i` 个节点没有下一个更大的节点，设置 `answer[i] = 0` 。

---

换成数组就行

```go
func nextLargerNodes(head *ListNode) []int {
    src := make([]int, 0)
    for tmp := head; tmp != nil; tmp = tmp.Next {
        src = append(src, tmp.Val)
    }
    st := make([]int, 0)
    ans := make([]int, len(src))
    for i, x := range src {
        for len(st) > 0 && src[st[len(st) - 1]] < x {
            idx := st[len(st) - 1]
            st = st[:len(st) - 1]
            ans[idx] = x
        }
        st = append(st, i)
    }
    return ans
}
```

