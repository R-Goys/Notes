[654. 最大二叉树](https://leetcode.cn/problems/maximum-binary-tree/)

> 给定一个不重复的整数数组 `nums` 。 **最大二叉树** 可以用下面的算法从 `nums` 递归地构建:
>
> 1. 创建一个根节点，其值为 `nums` 中的最大值。
> 2. 递归地在最大值 **左边** 的 **子数组前缀上** 构建左子树。
> 3. 递归地在最大值 **右边** 的 **子数组后缀上** 构建右子树。
>
> 返回 *`nums` 构建的* ***最大二叉树\*** 。

---

就是递归构造二叉树，感觉可以用栈优化？

```go
func constructMaximumBinaryTree(nums []int) *TreeNode {
    if len(nums) == 0 {
        return nil
    }
    maxn := 0
    for i, x := range nums {
        if x > nums[maxn] {
            maxn = i
        }
    }
    return &TreeNode{
        Val: nums[maxn],
        Left: constructMaximumBinaryTree(nums[:maxn]),
        Right: constructMaximumBinaryTree(nums[maxn + 1:]),
    }
}
```

