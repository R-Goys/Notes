[445. 两数相加 II](https://leetcode.cn/problems/add-two-numbers-ii/)

> 给你两个 **非空** 链表来代表两个非负整数。数字最高位位于链表开始位置。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。
>
> 你可以假设除了数字 0 之外，这两个数字都不会以零开头。

---

不需要反转链表，用栈将节点存入即可，注意进位的计算。

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
	stk1 := make([]*ListNode, 0)
	stk2 := make([]*ListNode, 0)
	for tmp := l1; tmp != nil; tmp = tmp.Next {
		stk1 = append(stk1, tmp)
	}
	for tmp := l2; tmp != nil; tmp = tmp.Next {
		stk2 = append(stk2, tmp)
	}
	if len(stk1) < len(stk2) {
		stk1, stk2 = stk2, stk1
	}
	pre, cur := &ListNode{}, &ListNode{}

	for len(stk1) != 0 && len(stk2) != 0 {
		node1 := stk1[len(stk1) - 1]
		stk1 = stk1[:len(stk1) - 1]
		node2 := stk2[len(stk2) - 1]
		stk2 = stk2[:len(stk2) - 1]
		num := (cur.Val + node1.Val + node2.Val) / 10
		cur.Val = (cur.Val + node1.Val + node2.Val) % 10
		pre = cur
		cur = &ListNode{}
		cur.Next = pre
		cur.Val += num
	}
	for len(stk1) != 0 {
		node1 := stk1[len(stk1) - 1]
		stk1 = stk1[:len(stk1) - 1]
        num := (cur.Val + node1.Val) / 10
		cur.Val = (cur.Val + node1.Val) % 10
		pre = cur
		cur = &ListNode{}
		cur.Next = pre
        cur.Val += num
	}
	if cur.Val != 0 {
		return cur
	}
	return cur.Next
}
```

