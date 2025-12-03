[83. 删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)

> 给定一个已排序的链表的头 `head` ， *删除所有重复的元素，使每个元素只出现一次* 。返回 *已排序的链表* 。

虚拟头节点保存头(这是一个好习惯)，然后遍历，如果遇见当前节点和下一个节点的值相同，则将当前节点的下一个节点指向下下个节点。

```go
func deleteDuplicates(head *ListNode) *ListNode {
    dummy := &ListNode{
        Next: head,
        Val: 101,
    }
    tmp := dummy.Next
    for tmp != nil && tmp.Next != nil {
        if tmp.Val == tmp.Next.Val {
            tmp.Next = tmp.Next.Next
        } else {
            tmp = tmp.Next
        }
    }
    return dummy.Next
}
```

