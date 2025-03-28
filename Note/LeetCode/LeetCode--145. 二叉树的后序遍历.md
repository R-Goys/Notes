[145. 二叉树的后序遍历](https://leetcode.cn/problems/binary-tree-postorder-traversal/)

> 给你一棵二叉树的根节点 `root` ，返回其节点值的 **后序遍历** 。

从简单的递归开始：

```go
func postorderTraversal(root *TreeNode) []int {
    var ans []int
    postorder(root, &ans)
    return ans
}

func postorder(node *TreeNode, ans *[]int) {
    if node == nil {
        return
    }
    postorder(node.Left, ans)
    postorder(node.Right, ans)
    *ans = append(*ans, node.Val)
}
```

非递归：

又写了一遍，还是不熟悉，我发现前中后序遍历并不是直接从栈中拿出元素直接进行遍历的，虽然就是这样，但是也用了一个指针去跟踪我们的遍历的节点，记住这一点，应该就比较好思考了。

```go
func postorderTraversal(root *TreeNode) []int {
    var stk []*TreeNode
    p := root
    var ans []int
    var prev *TreeNode
    for len(stk) > 0 || p != nil {
        for p != nil {
            stk = append(stk, p)
            p = p.Left
        }

        p = stk[len(stk) - 1]
        stk = stk[:len(stk) - 1]
        if p.Right == nil || p.Right == prev {
            ans = append(ans, p.Val)
            prev = p
            p = nil
        } else {
            stk = append(stk, p)
            p = p.Right
        }
    }
    return ans
}
```

