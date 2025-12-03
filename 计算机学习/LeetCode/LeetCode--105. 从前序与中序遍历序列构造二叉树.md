[105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

> 给定两个整数数组 `preorder` 和 `inorder` ，其中 `preorder` 是二叉树的**先序遍历**， `inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

---

递归构造，利用前序遍历和中序遍历的性质来做就是比较简单的题:

```go
func buildTree(preorder []int, inorder []int) *TreeNode {
    Index := 0
    return GetTree(preorder, inorder, &Index, 0, len(preorder) - 1)
}

func GetTree(preorder []int, inorder []int, PreIndex *int, Left int, Right int) *TreeNode {
    if Left > Right {
        return nil
    }
    Mid := Left
    for Mid = Left; Mid <= Right; Mid ++ {
        if inorder[Mid] == preorder[*PreIndex] {
            break
        }
    }
    Node := &TreeNode{Val: preorder[*PreIndex]}
    (*PreIndex) ++
    Node.Left = GetTree(preorder, inorder, PreIndex, Left, Mid - 1)
    Node.Right = GetTree(preorder, inorder, PreIndex, Mid + 1, Right)
    return Node
}
```

