[543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)

> 给你一棵二叉树的根节点，返回该树的 **直径** 。
>
> 二叉树的 **直径** 是指树中任意两个节点之间最长路径的 **长度** 。这条路径可能经过也可能不经过根节点 `root` 。
>
> 两节点之间路径的 **长度** 由它们之间边数表示。

----

这道题还是比较简单的

暴搜两侧，向上返回当前子树最长的一条路径

再利用一个ans变量存储最大的直径

```go
func diameterOfBinaryTree(root *TreeNode) int {
    var ans int
    dfs(root, &ans)
    return ans
}

func dfs(node *TreeNode, ans *int) int {
    if node == nil {
        return 0
    }
    L := dfs(node.Left, ans)
    R := dfs(node.Right, ans)
    *ans = max(*ans, L + R)
    return max(L, R) + 1
}
```

