[124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)

> 二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。
>
> **路径和** 是路径中各节点值的总和。
>
> 给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。

----

### dfs

比较容易想到的思路就是深度优先搜索，首先应该定义一个`ans`遍历，存储我们遍历到的最大值，然后深搜参数传递节点指针以及`ans`指针，在遍历中，我们需要考虑到所有的情况，在获取当前节点的最大值时，是要考虑到哪些值？

要考虑到的有：根节点，左儿子路径最大值 + 根节点，右儿子路径最大值 + 根节点， 以及左右儿子路径最大值的和 + 根节点，如此一来，是不是还是感觉太复杂了？这时，只需要在获取`LeftMax`和`RightMax`的时候和0取最大值，便可以不用担心负数的情况，可以直接处理。

但是我们还需要注意到，由于我们还需要获取左右儿子的路径最大值，所以我们递归向上传递的只能是当前路径总和的最大值，而不能是`Left + Right + Root.Val`，也不是`ans`，而是 `max(left, right) + Node.Val`

于是就有了：

```go
func maxPathSum(root *TreeNode) int {
    ans := root.Val
    dfs(root, &ans)
    return ans
}

func dfs(Node *TreeNode, ans *int) int {    
    if Node == nil {
        return 0
    }

    left := max(dfs(Node.Left, ans), 0)
    right := max(dfs(Node.Right, ans), 0)

    *ans = max(*ans, left + right + Node.Val)

    return max(left, right) + Node.Val
}
```



