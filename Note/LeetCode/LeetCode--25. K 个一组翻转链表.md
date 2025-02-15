[25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

>给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。
>
>`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。
>
>你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

---

这道题的题目可以近似为将n个链表反转并拼接起来。

虽然给定的链表是拼接好了的，但是每一段链表反转之后依旧少不了拼接的这一部分，大体的思路如下：

循环遍历链表中的元素，每k个节点为一组，用head和tail来维护这段链表，然后需要一个变量prev来存储head的前面一个节点是什么，在链表反转操作时候，此时，tail相当于这段链表的head，所以tail就可以作为下一段链表的prev，由此来进行找到下一段链表的head和tail

反转链表的部分就近似于[206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)这道题，有需要可以看看我的[Github题解](https://github.com/Rinai-R/Notes/blob/main/Note/LeetCode/LeetCode--206.%20%E5%8F%8D%E8%BD%AC%E9%93%BE%E8%A1%A8.md)

下面看看代码：

```go
func reverseKGroup(head *ListNode, k int) *ListNode {
    dummy := &ListNode{
        Next: head,
    }
    prev := dummy
    for {
        tail := prev
        for i := 0; i < k; i ++ {
            tail = tail.Next
            if tail == nil {
                //无需继续遍历进行修改，直接返回头节点
                return dummy.Next
            }
        }
        ne := tail.Next
        head, tail = Reverse(head, tail)
        prev.Next = head
        tail.Next = ne
        prev = tail
        head = tail.Next
    }
    return dummy.Next
}

//反转链表的内容
func Reverse(head *ListNode, tail *ListNode) (*ListNode, *ListNode) {
    prev := tail.Next
    cur := head
    for prev != tail {
        ne := cur.Next
        cur.Next = prev
        prev = cur
        cur = ne
    }
    return tail, head
}
```

