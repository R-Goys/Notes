[958. 二叉树的完全性检验](https://leetcode.cn/problems/check-completeness-of-a-binary-tree/)

> 给你一棵二叉树的根节点 `root` ，请你判断这棵树是否是一棵 **完全二叉树** 。
>
> 在一棵 **[完全二叉树](https://baike.baidu.com/item/完全二叉树/7773232?fr=aladdin)** 中，除了最后一层外，所有层都被完全填满，并且最后一层中的所有节点都尽可能靠左。最后一层（第 `h` 层）中可以包含 `1` 到 `2h` 个节点。

---

没想出来，原来给的样例已经有很多提示了，就是为每个节点都打上标记，使得当前节点的数字是父节点的2倍或者2倍+1

最后看看最后一个数字的大小是否与节点的数量一致。

```go
func isCompleteTree(root *TreeNode) bool {
    q := make([]Anode,0)
    q = append(q, Anode{root, 1})
    i := 0
    for i < len(q) {
        top := q[i]
        i ++
        if top.node != nil {
            q = append(q, Anode{top.node.Left, top.num*2})
            q = append(q, Anode{top.node.Right, top.num*2 + 1})
        }
    }
    return len(q) == q[len(q) - 1].num
}

type Anode struct{
    node *TreeNode
    num int
}
```

