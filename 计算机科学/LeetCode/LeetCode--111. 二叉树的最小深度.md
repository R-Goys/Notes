[111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)

> 给定一个二叉树，找出其最小深度。
>
> 最小深度是从根节点到最近叶子节点的最短路径上的节点数量。
>
> **说明：**叶子节点是指没有子节点的节点。

---

面试不会这么简单，秒了

```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    ans := 10000
    dfs(root, &ans, 1)
    return ans
}

func dfs(root *TreeNode, ans *int, depth int) {
    if root.Left == nil && root.Right == nil {
        *ans = min(*ans, depth)
        return
    }
    if root.Left != nil {
        dfs(root.Left, ans, depth + 1)
    }
    if root.Right != nil {
        dfs(root.Right, ans, depth + 1)
    }
}
```

