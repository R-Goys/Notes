[450. 删除二叉搜索树中的节点](https://leetcode.cn/problems/delete-node-in-a-bst/)

> 给定一个二叉搜索树的根节点 **root** 和一个值 **key**，删除二叉搜索树中的 **key** 对应的节点，并保证二叉搜索树的性质不变。返回二叉搜索树（有可能被更新）的根节点的引用。
>
> 一般来说，删除节点可分为两个步骤：
>
> 1. 首先找到需要删除的节点；
> 2. 如果找到了，删除它。

---

递归删除，最主要的就是要直到节点是怎么再次拼接的，比AVL树简单很多，没那么复杂。

但是这里的拼接二叉树的思路确实巧妙，我最开始想的是直接像链表那种删除，反而更麻烦了，这种递归拼接反而更简单。

```go
func deleteNode(root *TreeNode, key int) *TreeNode {
    switch {
        case root == nil:
            return nil
        case root.Val < key:
            root.Right = deleteNode(root.Right, key)
        case root.Val > key:
            root.Left = deleteNode(root.Left, key)
        case root.Left == nil || root.Right == nil: 
            if root.Left != nil {
                return root.Left
            }
            return root.Right
        default:
            successor := root.Right
            for successor.Left != nil {
                successor = successor.Left
            }
            successor.Right = deleteNode(root.Right, successor.Val)
            successor.Left = root.Left
            return successor
    }
    return root
}
```

