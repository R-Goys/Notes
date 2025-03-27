[328. 奇偶链表](https://leetcode.cn/problems/odd-even-linked-list/)

> 给定单链表的头节点 `head` ，将所有索引为奇数的节点和索引为偶数的节点分别组合在一起，然后返回重新排序的列表。
>
> **第一个**节点的索引被认为是 **奇数** ， **第二个**节点的索引为 **偶数** ，以此类推。
>
> 请注意，偶数组和奇数组内部的相对顺序应该与输入时保持一致。
>
> 你必须在 `O(1)` 的额外空间复杂度和 `O(n)` 的时间复杂度下解决这个问题。

---

简单，秒杀

```go
func oddEvenList(head *ListNode) *ListNode {
    if head == nil {
        return nil
    }
    dummy := &ListNode{Next:head}
    evenHead := head.Next
    odd, even := head, evenHead
    for even != nil && even.Next != nil {
        odd.Next = odd.Next.Next
        odd = odd.Next
        even.Next = even.Next.Next
        even = even.Next
    }
    odd.Next = evenHead
    return dummy.Next
}
```

