[103. 二叉树的锯齿形层序遍历](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal/)

> 给你二叉树的根节点 `root` ，返回其节点值的 **锯齿形层序遍历** 。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

----

层序遍历题加个反向即可

```go
func zigzagLevelOrder(root *TreeNode) [][]int {
    var ans [][]int
    Order(root, 0, &ans)
    for depth := 0; depth < len(ans); depth ++{
        if depth % 2 == 1 {
            for i, n := 0, len((ans)[depth]); i < n / 2; i ++ {
                (ans)[depth][i], (ans)[depth][n - i - 1] = (ans)[depth][n - i - 1], (ans)[depth][i]
            }
        }
    }
    return ans
}

func Order(Node *TreeNode, depth int, ans *[][]int) {
    if Node == nil {
        return
    }
    if len(*ans) <= depth {
        *ans = append(*ans, []int{})
    }
    (*ans)[depth] = append((*ans)[depth], Node.Val)
    Order(Node.Left, depth + 1, ans)
    Order(Node.Right, depth + 1, ans)

    return
}
```

