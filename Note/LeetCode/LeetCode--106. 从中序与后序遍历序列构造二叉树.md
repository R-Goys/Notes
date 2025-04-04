[106. 从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

> 给定两个整数数组 `inorder` 和 `postorder` ，其中 `inorder` 是二叉树的中序遍历， `postorder` 是同一棵树的后序遍历，请你构造并返回这颗 *二叉树* 。

---

这种根据两个序构造二叉树都挺模板的，没啥好说的，但是注意build的顺序一定要正确。

```go
func buildTree(inorder []int, postorder []int) *TreeNode {
    mp := make(map[int]int, 0)
    for i := 0; i < len(inorder); i ++ {
        mp[inorder[i]] = i
    }
    index := len(postorder) - 1
    return build(inorder, postorder, 0, len(inorder) - 1, mp, &index)
}

func build(inorder, postorder []int, left, right int, mp map[int]int, index *int) *TreeNode {
    if left > right{ 
        return nil
    }
    root := &TreeNode{
        Val: postorder[*index],
    }
    (*index) --
    mid := mp[root.Val]
    root.Right = build(inorder, postorder, mid + 1, right, mp, index)
    root.Left = build(inorder, postorder, left, mid - 1, mp, index)
    return root
}
```

