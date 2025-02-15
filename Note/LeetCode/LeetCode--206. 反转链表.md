[206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

>给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

----

麻了，感觉自己太不擅长链表的题目了，easy都卡了一下，这里只需要存储Next的节点就行，不需要存储ne.ne，这只会加剧复杂度，绕来绕去都给我绕晕了。

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
	var prev *ListNode
    cur := head

    for cur != nil {
        NextTmp := cur.Next
        cur.Next = prev
        prev = cur
        cur = NextTmp
    }
    return prev
}
```





递归做法：

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    return Reverse(nil, head)
}

func Reverse(Prev *ListNode, Cur *ListNode) *ListNode {
    if Cur == nil {
        return Prev
    }
    Ne := Cur.Next
    Cur.Next = Prev
    return Reverse(Cur, Ne)
}
```

