[98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)

> 给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。
>
> **有效** 二叉搜索树定义如下：
>
> - 节点的左子树只包含 **小于** 当前节点的数。
> - 节点的右子树只包含 **大于** 当前节点的数。
> - 所有左子树和右子树自身必须也是二叉搜索树。

----

dfs

还有另一种方法是中序遍历，可以让二叉搜索树变成单调递增的数组。

```go
func isValidBST(root *TreeNode) bool {
    return dfs(root.Left, root.Val, math.MinInt) && dfs(root.Right, math.MaxInt, root.Val)
}

func dfs(node *TreeNode, Max int, Min int) bool {
    if node == nil {
        return true
    }
    if node.Val >= Max || node.Val <= Min {
        return false
    }
    return dfs(node.Left, node.Val, Min) && dfs(node.Right, Max, node.Val)
}
```

