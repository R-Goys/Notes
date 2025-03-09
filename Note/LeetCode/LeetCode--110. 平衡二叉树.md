[110. 平衡二叉树](https://leetcode.cn/problems/balanced-binary-tree/)

> 给定一个二叉树，判断它是否是 平衡二叉树 

----

直接递归开搜

```go
func isBalanced(root *TreeNode) bool {
    ok := true
    solve(root, &ok)
    return ok
}

func solve(node *TreeNode, ok *bool) int {
    if node == nil {
        return 0
    }
    L := solve(node.Left, ok)
    R := solve(node.Right, ok)
    if abs(L - R) > 1 {
        *ok = false
    }
    return max(L, R) + 1
}

func abs(num int) int {
    if num > 0 {
        return num
    }
    return -num
}
```

事实上，这个代码还可以进行剪枝，毕竟一旦判断了当前bool值已经为false了，就没必要继续判断下去了。

```go
func isBalanced(root *TreeNode) bool {
    ok := true
    solve(root, &ok)
    return ok
}

func solve(node *TreeNode, ok *bool) int {
    if *ok == false {
        return 0
    }
    if node == nil {
        return 0
    }
    L := solve(node.Left, ok)
    R := solve(node.Right, ok)
    if abs(L - R) > 1 {
        *ok = false
    }
    return max(L, R) + 1
}

func abs(num int) int {
    if num > 0 {
        return num
    }
    return -num
}
```

又去看了官方的做法，如果判断当前子树不平衡，直接将返回的int设置为-1，还省去了bool值的麻烦。