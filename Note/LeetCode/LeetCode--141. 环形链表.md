[141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

> 给你一个链表的头节点 `head` ，判断链表中是否有环。
>
> 如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。
>
> *如果链表中存在环* ，则返回 `true` 。 否则，返回 `false` 。

----

## 快慢指针

```go
func hasCycle(head *ListNode) bool {

    node1 := &ListNode{Next:head}
    node2 := head
    for node1 != node2 && node1 != nil && node2 != nil {
        node1 = node1.Next
        node2 = node2.Next
        if node2 != nil {
            node2 = node2.Next
        } else {
            break
        }
    }
    if node1 == nil || node2 == nil {
        return false
    } else {
        return true
    }
}
```

## 哈希表

```go
func hasCycle(head *ListNode) bool {
    mp := make(map[*ListNode]bool, 1)
    for node := head; node != nil; node = node.Next {
        if mp[node] {
            return true
        }
        mp[node] = true
    }
    return false
}
```

个人还是喜欢双指针，优雅，哈希表有点内啥了