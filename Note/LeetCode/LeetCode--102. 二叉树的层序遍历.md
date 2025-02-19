[102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

>给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

----

递归找节点，将用depth确定加入二维数组的哪个位置，直接将node的val加入ans即可。

```go
func levelOrder(root *TreeNode) [][]int {
    var ans [][]int
    LevelOrder(root, 0, &ans)
    return ans
}

func LevelOrder(Node *TreeNode, depth int, ans *[][]int) {
    if Node == nil {
        return
    }
    if len(*ans) <= depth {
        *ans = append(*ans, []int{})
    }
    (*ans)[depth] = append((*ans)[depth], Node.Val)
    LevelOrder(Node.Left, depth + 1, ans)
    LevelOrder(Node.Right, depth + 1, ans)
    return
}
```

