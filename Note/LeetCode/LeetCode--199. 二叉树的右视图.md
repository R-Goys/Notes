[199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)

> 给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

---

比较简单的一道题，用一个布尔数组维护遍历的深度，每次优先遍历右节点，若没有遍历过当前深度，则直接加入ans即可

```go
func rightSideView(root *TreeNode) []int {
    st := make([]bool, 100)
    ans := make([]int, 0)
    dfs(root, &st, &ans, 0)
    return ans
}

func dfs(Node *TreeNode, st *[]bool, ans *[]int, depth int) {
    if Node == nil {
        return
    }
    if !(*st)[depth] {
        *ans = append(*ans, Node.Val)
        (*st)[depth] = true
    }
    dfs(Node.Right, st, ans, depth + 1)
    dfs(Node.Left, st, ans, depth + 1)
}
```

