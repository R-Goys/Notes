[24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

> 给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

---

简单的链表变换。

```go
func swapPairs(head *ListNode) *ListNode {
    dummy := &ListNode{
        Next:head,
    }
    prev := dummy
    for prev.Next != nil && prev.Next.Next != nil {
        fir := prev.Next
        sec := prev.Next.Next
        prev.Next = sec
        fir.Next = sec.Next
        sec.Next = fir

        prev = fir
    }
    return dummy.Next
}
```

