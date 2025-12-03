[61. 旋转链表](https://leetcode.cn/problems/rotate-list/)

> 给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置。

注意k为0或者为链表长度的整数倍的时候，需要特判

```go
func rotateRight(head *ListNode, k int) *ListNode {
    if head == nil {
        return head
    }
    n := 0
    for tmp := head; tmp != nil; tmp = tmp.Next {
        n ++
    }
    k %= n
    if k == 0 {
        return head
    }
    dummy := &ListNode{Next:head}
    ne, cur := dummy, dummy
    for i := 0; i < k; i ++ {
        ne = ne.Next
    }
    for ne.Next != nil {
        ne = ne.Next
        cur = cur.Next
    }
    dummy.Next = cur.Next
    ne.Next = head
    cur.Next = nil
    return dummy.Next
}
```

