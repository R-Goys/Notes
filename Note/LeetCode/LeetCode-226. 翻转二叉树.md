[226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

> 给你一棵二叉树的根节点 `root` ，翻转这棵二叉树，并返回其根节点。

要是面试有这么简单就好了

递归

```go
func invertTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }
    root.Right, root.Left = root.Left, root.Right
    invertTree(root.Right)
    invertTree(root.Left)
    return root
}
```

迭代

```go
func invertTree(root *TreeNode) *TreeNode {
    q := make([]*TreeNode, 0)
    q = append(q, root)
    for len(q) > 0 {
        top := q[len(q) - 1]
        q = q[:len(q) - 1]
        if top == nil {
            continue
        }
        top.Right, top.Left = top.Left, top.Right
        q = append(q, top.Right)
        q = append(q, top.Left)
    }
    return root
}
```

