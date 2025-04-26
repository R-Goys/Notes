[337. 打家劫舍 III](https://leetcode.cn/problems/house-robber-iii/)

> 小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为 `root` 。
>
> 除了 `root` 之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果 **两个直接相连的房子在同一天晚上被打劫** ，房屋将自动报警。
>
> 给定二叉树的 `root` 。返回 ***在不触动警报的情况下** ，小偷能够盗取的最高金额* 。

---

在二叉树上的dp还没试过。。

使用哈希表来存储，每个节点的最大贡献值，使用两个哈希表表示两种状态，一种是选择当前节点，一种是不选择当前节点：

```go
var f map[*TreeNode]int
var g map[*TreeNode]int

func rob(root *TreeNode) int {
    f = make(map[*TreeNode]int)
    g = make(map[*TreeNode]int)
    dfs(root)
    return max(f[root], g[root])
}

func dfs(node *TreeNode) {
    if node == nil {
        return
    }

    dfs(node.Left)
    dfs(node.Right)

    f[node] = node.Val + g[node.Left] + g[node.Right]
    g[node] = max(f[node.Left], g[node.Left]) + max(f[node.Right], g[node.Right])
}
```

