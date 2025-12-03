[92. 反转链表 II](https://leetcode.cn/problems/reverse-linked-list-ii/)

> 给你单链表的头指针 `head` 和两个整数 `left` 和 `right` ，其中 `left <= right` 。请你反转从位置 `left` 到位置 `right` 的链表节点，返回 **反转后的链表** 。

---

就是反转链表变个样子,值得关注的点就是start和end的选取.

```go
func reverseBetween(head *ListNode, left int, right int) *ListNode {
    dummy := &ListNode{Next: head}
    Prev := dummy
    Tail := dummy
    for i := 1; i < left; i ++ {
        Prev = Prev.Next
    }
    for i := 1; i <= right; i ++ {
        Tail = Tail.Next
    }
    Head := Prev.Next
    Ne := Tail.Next
    
    Reverse(Head, Ne)
    Prev.Next = Tail
    Head.Next = Ne
    return dummy.Next 
}

func Reverse(start *ListNode, end *ListNode){
    tmp := start
    var prev *ListNode = nil
    for tmp != end {
        Ne := tmp.Next
        tmp.Next = prev
        prev = tmp
        tmp = Ne
    }
    return
}
```

