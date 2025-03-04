[LCR 140. 训练计划 II](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

> 给定一个头节点为 `head` 的链表用于记录一系列核心肌群训练项目编号，请查找并返回倒数第 `cnt` 个训练项目编号。

----

一分钟秒了,前不久才做过，应该是时间复杂度和空间复杂度最优的做法了

```go
func trainingPlan(head *ListNode, cnt int) *ListNode {
    Node := head
    tail := head
    for i := 0; i < cnt; i ++ {
        tail = tail.Next
    }

    for tail != nil {
        Node = Node.Next
        tail = tail.Next
    }
    return Node
}
```

