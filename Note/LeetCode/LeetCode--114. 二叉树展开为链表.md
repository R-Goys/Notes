[114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)

> 给你二叉树的根结点 `root` ，请你将它展开为一个单链表：
>
> - 展开后的单链表应该同样使用 `TreeNode` ，其中 `right` 子指针指向链表中下一个结点，而左子指针始终为 `null` 。
> - 展开后的单链表应该与二叉树 [**先序遍历**](https://baike.baidu.com/item/先序遍历/6442839?fr=aladdin) 顺序相同。

---

栈：

```go
func flatten(root *TreeNode)  {
    if root == nil {
        return
    }
    tmp := &TreeNode{}
    q := make([]*TreeNode, 0)
    q = append(q, root)
    for len(q) != 0 {
        node := q[len(q) - 1]
        q = q[:len(q) - 1]
        tmp.Right = node
        tmp.Left = nil
        tmp = node
        if node.Right != nil {
            q = append(q, node.Right)
        }
        if node.Left != nil {
            q = append(q, node.Left)
        }
    }
}
```

存前驱节点

这个思路有意思，每一次遍历有父子两个节点，先找到子节点的右节点的最后一个元素，然后将其父节点的右儿子拼接到这个元素上面，然后移动当前节点(父节点)调整一一下左右节点，随后移动到他的下一个节点上，继续遍历，如果遇到了当前节点存在左儿子就像刚刚一样展开，如果没有，就说明当前链表不需要修改。

```go
func flatten(root *TreeNode)  {
    cur := root
    for cur != nil {
        if cur.Left != nil {
            next := cur.Left
            predecessor := next
            for predecessor.Right != nil {
                predecessor = predecessor.Right
            }
            predecessor.Right = cur.Right
            cur.Left, cur.Right = nil, next
        }
        cur = cur.Right
    }
}
```

