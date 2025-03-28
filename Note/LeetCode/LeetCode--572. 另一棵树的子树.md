[572. 另一棵树的子树](https://leetcode.cn/problems/subtree-of-another-tree/)

> 给你两棵二叉树 `root` 和 `subRoot` 。检验 `root` 中是否包含和 `subRoot` 具有相同结构和节点值的子树。如果存在，返回 `true` ；否则，返回 `false` 。
>
> 二叉树 `tree` 的一棵子树包括 `tree` 的某个节点和这个节点的所有后代节点。`tree` 也可以看做它自身的一棵子树。

---

判断与搜索分开，避免判断错误

这样，在判断时，保证subRoot不变，而root可变，这样可以在任意位置验证是否为子树。

```go
func isSubtree(root *TreeNode, subRoot *TreeNode) bool {
    if subRoot == nil {
        return true
    }
    if root == nil {
        return false
    }

    if solve(root, subRoot) {
        return true
    }
    return isSubtree(root.Right, subRoot) || isSubtree(root.Left, subRoot)
}

func solve(root *TreeNode, subRoot *TreeNode) bool {
    if subRoot == nil && root == nil {
        return true
    }
    if root == nil || subRoot == nil {
        return false
    }

    return root.Val == subRoot.Val && solve(root.Left, subRoot.Left) && solve(root.Right, subRoot.Right)
}
```

