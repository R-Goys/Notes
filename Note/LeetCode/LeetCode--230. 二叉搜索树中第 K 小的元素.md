[230. 二叉搜索树中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)

> 给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 小的元素（从 1 开始计数）。

---

暴搜的做法：

```go
func kthSmallest(root *TreeNode, k int) int {
        ans, cur := 0, 0
        dfs(root, k, &cur, &ans)
        return ans
}

func dfs(node *TreeNode, k int, cur, ans *int) {
        if node == nil {
                return
        }
        dfs(node.Left, k, cur, ans)
        *cur ++
        if *cur == k {
                *ans = node.Val
        }
        dfs(node.Right, k, cur, ans)
}
```



迭代，相当于剪枝了，搜索到就直接返回，无需多余遍历：

```go
func kthSmallest(root *TreeNode, k int) (ans int) {
        var stk []*TreeNode
        for len(stk) != 0 || root != nil {
                for root != nil {
                        stk = append(stk, root)
                        root = root.Left
                }
                root = stk[len(stk) - 1]
                stk = stk[:len(stk) - 1]
                k --
                if k == 0 {
                        ans = root.Val
                        return
                }
                root = root.Right
        }
        return
}
```



