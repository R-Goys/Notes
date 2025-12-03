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

morris遍历，带注释版本，感觉面试是有极大的可能撕不出来的

但是最好还是写一遍，然后自己画图理解一遍，至少面试问到这玩意，尽管真的没撕出来，至少能吹一下)

```go
func kthSmallest(root *TreeNode, k int) int {
        return morris(root, k)
}

func morris(root *TreeNode, k int) (ans int) {
        for root != nil {
                if root.Left != nil {
                        //如果当前节点有左子树
                        //那么我们需要在这里为当前节点和他的前驱节点建立连接
                        //以便于我们实现morris遍历
                        pre := root.Left
                        //找到左子树的最右节点
                        for pre.Right != nil && pre.Right != root {
                                pre = pre.Right
                        }
                        //搜索完毕,更新pre的下一个节点.
                        if pre.Right == nil {
                                pre.Right = root
                                root = root.Left
                        } else {
                                //这个分支说明这是第二次访问当前节点,此时是
                                //真正的在遍历的状态,正常执行我们的
                                //中序遍历逻辑就可以了
                                k--
                                if k == 0 {
                                        ans = root.Val
                                        return
                                }
                                //恢复我们的树结构
                                pre.Right = nil
                                root = root.Right
                        }
                } else {
                        //说明没有左子树
                        //直接访问当前节点即可.
                        k--
                        if k == 0 {
                                ans = root.Val
                                return
                        }
                        root = root.Right
                }
        }
        return
}
```

