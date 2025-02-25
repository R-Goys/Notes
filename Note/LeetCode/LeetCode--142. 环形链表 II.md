[142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

> 给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*
>
> 如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。
>
> **不允许修改** 链表。

----

一刷，二刷都没独立写出来，在这里写个题解，加深记忆

### 双指针

此处最初的思路和[141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)一致，首先要判断出是否存在环形链表，然后再去寻找环入口节点，通过快慢指针fast，slow首先指向head，随后fast每次比slow多走一步，如果fast == nil 则说明不存在环形链表，直接返回nil，如果fast == slow 那么存在环形链表，进行下一步计算：

既然此时已经相遇，设fast在环中走过n圈，与环入口的距离为x，设环外的长度为y，环长为m，则有：

`n * m + x + y = 2 * (x + y)`，所以推出了：y = m * n - x

即环外的长度，正好等于n圈环长 + x，也就是说，如果我们再定义一个从头节点ptr开始走，让slow指针继续以相同的速度走，当ptr走完y的距离，也就是到达环入口时，slow恰好走了n圈少走了x的距离，也就是走到了换入口，此时，相遇，便可以输出我们的环入口节点了。

```go
func detectCycle(head *ListNode) *ListNode {
    slow, fast := head, head
    for fast != nil {
        slow = slow.Next
        fast = fast.Next
        if fast == nil {
            return nil
        }
        fast = fast.Next
        if fast == slow {
            ptr := head
            for ptr != slow {
                ptr = ptr.Next
                slow = slow.Next
            }
            return ptr
        }
    }
    return nil
}
```

