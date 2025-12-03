[234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)

> 给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。

----

空间O(1)的做法这么麻烦，你管他叫easy？

首先O(N)的做法是很简单的，比如栈，数组之类的

但是O(1)就稍微有一点麻烦，涉及到链表的反转，恢复

首先用快慢指针找到中间节点，然后通过中间节点将后面的链表全部反转，然后开始比较，只要一遇见不同的值，立刻返回false，否则继续遍历，直到遇到nil，此时我们可以判断，这个链表是回文链表，但是通常我们不能直接修改这个传入的链表的结构，所以我们还需要恢复这个链表，直接再使用一次链表反转即可，同时我们之前还需要存储一个pre变量，保存mid前面一个节点，方便把恢复后的链表连接起来。

```go
func isPalindrome(head *ListNode) bool {
    mid, tail := head, head
    prev := mid
    for tail != nil {
        prev = mid
        mid = mid.Next
        tail = tail.Next
        if tail != nil {
            tail = tail.Next
        }
    }
    start := ReverseList(mid, nil)
    tmp := start
    tmp2 := head
    for tmp != nil && tmp2 != nil {
        if tmp.Val != tmp2.Val {
            return false
        }
        tmp = tmp.Next
        tmp2= tmp2.Next
    }
    ReverseList(tail, nil)
    prev.Next = mid
    return true

}

func ReverseList(head, tail *ListNode) *ListNode {
    var prev *ListNode = nil
    tmp := head
    for tmp != tail {
        Ne := tmp.Next
        tmp.Next = prev
        prev = tmp
        tmp = Ne
    }
    return prev
}
```

