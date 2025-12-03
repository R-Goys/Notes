[82. 删除排序链表中的重复元素 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/)

> 给定一个已排序的链表的头 `head` ， *删除原始链表中所有重复数字的节点，只留下不同的数字* 。返回 *已排序的链表* 。

----

基本思路是遍历Node，当Node.Val != Node.Next.Val时，存储当前Node作为Pre节点，如果相等，存储当前Node的Val作为PreVal，之后若满足Node.Val == PreVal，或者Node.Val == Node.Next.Val，即满足了重复特点，最后一旦不满足条件，令Pre.Next = 当前的Node即可，值得注意的是，如果重复的元素在链表末尾，无法在循环中就直接完成消除的过程，所以在循环结束的时候还需要计算一次Pre.Next = Node，代码如下：

```go
func deleteDuplicates(head *ListNode) *ListNode {
    dummy := &ListNode{Next:head}
    var PreVal = -114514
    tmp, pre := dummy.Next, dummy
    for ; tmp != nil; tmp = tmp.Next {
        if (tmp.Next != nil && tmp.Val == tmp.Next.Val) || tmp.Val == PreVal {
            PreVal = tmp.Val
            continue
        } else {
            pre.Next = tmp
            pre = tmp
        }
    }
    pre.Next = tmp

    return dummy.Next
}
```

