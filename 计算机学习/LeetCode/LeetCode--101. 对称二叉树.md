[101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

> 给你一个二叉树的根节点 `root` ， 检查它是否轴对称。

---

### dfs

简单暴搜，感觉用非递归也能做，但是感觉有点麻烦呢

```go
func isSymmetric(root *TreeNode) bool {
    return solve(root.Left, root.Right)
}

func solve(Left *TreeNode, Right *TreeNode) bool {
    if Left == nil && Right == nil {
        return true
    } else if Left == nil || Right == nil {
        return false
    }

    if Left.Val != Right.Val {
        return false
    }

    return solve(Left.Right, Right.Left) && solve(Left.Left, Right.Right)
}
```

总体思路就是用两颗子树分别用反向的遍历方法即可得出答案。

### 迭代

两个栈存储左右子树的遍历节点，网上的题解大多都是用一个栈，我还是觉得这样关系比较分明一点.

```go
func isSymmetric(root *TreeNode) bool {
    LST := []*TreeNode{}
    RST := []*TreeNode{}
    LST = append(LST, root.Left)
    RST = append(RST, root.Right)
    for len(LST) != 0 && len(RST) != 0{
        L := LST[len(LST) - 1]
        LST = LST[:len(LST) - 1]
        R := RST[len(RST) - 1]
        RST = RST[:len(RST) - 1]
        if L == nil && R == nil {
            continue
        }
        if L == nil || R == nil {
            return false
        }
        if L.Val != R.Val {
            return false
        }
        LST = append(LST, L.Left)
        RST = append(RST, R.Right)

        LST = append(LST, L.Right)
        RST = append(RST, R.Left)
    }
    return true
}
```

