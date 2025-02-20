[143. 重排链表](https://leetcode.cn/problems/reorder-list/)

> 给定一个单链表 `L` 的头节点 `head` ，单链表 `L` 表示为：
>
> ```
> L0 → L1 → … → Ln - 1 → Ln
> ```
>
> 请将其重新排列后变为：
>
> ```
> L0 → Ln → L1 → Ln - 1 → L2 → Ln - 2 → …
> ```
>
> 不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

---

乍一看很复杂，其实就是几个easy题合在一起而已

首先是快慢指针，找到中间节点，然后给定的链表拆分为两个链表，最后反转第二个链表，交替插入链表即可。

```go
func reorderList(head *ListNode)  {
    if head.Next == nil {
        return
    }
    dummy := &ListNode{Next:head}
    slow := dummy
    fast := dummy
    for fast.Next != nil {
        slow = slow.Next
        fast = fast.Next
        if fast.Next != nil {
            fast = fast.Next
        }
    }
    Reverse(slow.Next, fast.Next)
    slow.Next = nil
    ans := dummy
    list1 := dummy.Next
    list2 := fast
    for list1 != nil && list2 != nil {
        ans.Next = list1
        list1 = list1.Next
        ans = ans.Next

        ans.Next = list2
        list2 = list2.Next
        ans = ans.Next
    }
    if list1 != nil {
        ans.Next = list1
    }
    if list2 != nil {
        ans.Next = list2
    }
}

func Reverse(head, tail *ListNode) {
    var prev *ListNode = nil
    for head != tail {
        ne := head.Next
        head.Next = prev
        prev = head
        head = ne
    }
}
```

