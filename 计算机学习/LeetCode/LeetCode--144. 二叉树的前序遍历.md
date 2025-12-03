[144. 二叉树的前序遍历](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

> 给你二叉树的根节点 `root` ，返回它节点值的 **前序** 遍历。

----

### 递归

```go
func preorderTraversal(root *TreeNode) []int {
    var ans []int
    solve(root, &ans)
    return ans
}

func solve(root *TreeNode, ans *[]int) {
    if root == nil {
        return
    }
    *ans = append(*ans, root.Val)
    solve(root.Left, ans)
    solve(root.Right, ans)
}
```

### 非递归

```go
func preorderTraversal(root *TreeNode) []int {
    var ans []int
    var Nodes []*TreeNode
    Nodes = append(Nodes, root)
    for root != nil && len(Nodes) > 0 {
        root = Nodes[len(Nodes) - 1]
        Nodes = Nodes[:len(Nodes) - 1]

        ans = append(ans, root.Val)

        if root.Right != nil {
            Nodes = append(Nodes, root.Right)
        }

        if root.Left != nil {
            Nodes = append(Nodes, root.Left)
        }
    }
    return ans
}
```

