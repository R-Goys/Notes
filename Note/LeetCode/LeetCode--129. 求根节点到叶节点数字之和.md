[129. 求根节点到叶节点数字之和](https://leetcode.cn/problems/sum-root-to-leaf-numbers/)

> 给你一个二叉树的根节点 `root` ，树中每个节点都存放有一个 `0` 到 `9` 之间的数字。
>
> 每条从根节点到叶节点的路径都代表一个数字：
>
> - 例如，从根节点到叶节点的路径 `1 -> 2 -> 3` 表示数字 `123` 。
>
> 计算从根节点到叶节点生成的 **所有数字之和** 。
>
> **叶节点** 是指没有子节点的节点。

----

一眼dfs，直接秒，不过我这个写法值得注意的地方就是在根节点为nil的时候处理情况会不一样，最开始我的思路就是遇到nil是，就将cur加入ans，但是实际上是错误的写法，因为根节点可能有一边不是nil，所以不能加入ans，于是就改成了下面这种，当两边都为nil的时候，才将cur加入答案，否则，继续dfs，同时也要在前面加上if node == nil来适当的剪枝。

```go
func sumNumbers(root *TreeNode) int {
    ans := 0
    dfs(root, &ans, 0)
    return ans
}

func dfs(node *TreeNode, ans *int, cur int) {
    if node == nil {
        return
    }
    cur = cur * 10 + node.Val
    if node.Left == nil && node.Right == nil {
        *ans += cur
    } else {
        dfs(node.Left, ans, cur)
        dfs(node.Right, ans, cur)
    }
    return
}

```

