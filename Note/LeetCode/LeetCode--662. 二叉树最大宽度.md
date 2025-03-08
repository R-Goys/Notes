[662. 二叉树最大宽度](https://leetcode.cn/problems/maximum-width-of-binary-tree/)

> 给你一棵二叉树的根节点 `root` ，返回树的 **最大宽度** 。
>
> 树的 **最大宽度** 是所有层中最大的 **宽度** 。
>
> 每一层的 **宽度** 被定义为该层最左和最右的非空节点（即，两个端点）之间的长度。将这个二叉树视作与满二叉树结构相同，两端点间会出现一些延伸到这一层的 `null` 节点，这些 `null` 节点也计入长度。
>
> 题目数据保证答案将会在 **32 位** 带符号整数范围内。

---

没写出来，主要的问题是不知道宽度咋计算，看了一遍题解自己敲了一遍：

关键就是首先需要去搜索左子树的每一层的最左节点的索引，并且存储，方便在搜索右子树的时候直接进行比较并得到最大值。

```go
func widthOfBinaryTree(root *TreeNode) int {
    LevelMin := map[int]int{}
    var dfs func(*TreeNode, int, int) int
    dfs = func(node *TreeNode, depth int, idx int) int {
        if node == nil {
            return 0
        }
        if _, ok := LevelMin[depth]; !ok {
            LevelMin[depth] = idx
        }
        L := dfs(node.Left, depth + 1, idx * 2)
        R := dfs(node.Right, depth + 1, idx * 2 + 1)
        return max(idx - LevelMin[depth] + 1, max(L, R))
    }
    return dfs(root, 1, 1)
}
```

