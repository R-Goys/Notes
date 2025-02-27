[2. 两数相加](https://leetcode.cn/problems/add-two-numbers/)

> 给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。
>
> 请你将两个数相加，并以相同形式返回一个表示和的链表。
>
> 你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

----

### 比高精度还简单

直接无脑暴力就行，用一个变量存储进位，然后依次相加，每次遍历创建一个新的节点来存储ans

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    dummy := &ListNode{}
    Ans := dummy
    Add := 0
    tmp1 := l1
    tmp2 := l2
    for tmp1 != nil || tmp2 != nil {
        if tmp1 != nil {
            Add += tmp1.Val
            tmp1 = tmp1.Next 
        }
        if tmp2 != nil {
            Add += tmp2.Val
            tmp2 = tmp2.Next
        }
        Ans.Next = &ListNode{Val:Add % 10}
        Ans = Ans.Next
        Add /= 10
    }
    if Add != 0 {
        Ans.Next = &ListNode{Val:Add % 10}
    }
    return dummy.Next
}
```

