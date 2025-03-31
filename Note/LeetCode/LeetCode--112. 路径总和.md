[112. 路径总和](https://leetcode.cn/problems/path-sum/)

> 给你二叉树的根节点 `root` 和一个表示目标和的整数 `targetSum` 。判断该树中是否存在 **根节点到叶子节点** 的路径，这条路径上所有节点值相加等于目标和 `targetSum` 。如果存在，返回 `true` ；否则，返回 `false` 。
>
> **叶子节点** 是指没有子节点的节点。

---

好多边界条件需要判断，还让我回忆了一下叶子结点的概念，写的时候直接搞错了叶子节点。

```go
func hasPathSum(root *TreeNode, targetSum int) bool {
    if root == nil {
        return false
    }
    return dfs(root, targetSum, 0)
}

func dfs(node *TreeNode, targetSum, preSum int) bool {
    if node.Right == nil && node.Left == nil {
        return targetSum == preSum + node.Val
    }

    if node.Left != nil && dfs(node.Left, targetSum, preSum + node.Val) {
        return true
    }

    if node.Right != nil && dfs(node.Right, targetSum, preSum + node.Val) {
        return true
    }

    return false
}
```

