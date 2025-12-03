[113. 路径总和 II](https://leetcode.cn/problems/path-sum-ii/)

> 给你二叉树的根节点 `root` 和一个整数目标和 `targetSum` ，找出所有 **从根节点到叶子节点** 路径总和等于给定目标和的路径。
>
> **叶子节点** 是指没有子节点的节点。

---

直接暴搜，easy.

```go
func pathSum(root *TreeNode, targetSum int) [][]int {
	if root == nil {
		return nil
	}
	var ans [][]int
	dfs(root, targetSum, &ans, []int{root.Val}, root.Val)
	return ans
}

func dfs(node *TreeNode, targetSum int, ans *[][]int, path []int, sum int) {
	if sum == targetSum && node.Left == nil && node.Right == nil {
		*ans = append(*ans, append([]int{}, path...))
	}
	if node == nil {
		return
	}

	if node.Left != nil {
		path = append(path, node.Left.Val)
		dfs(node.Left, targetSum, ans, path, sum+node.Left.Val)
		path = path[:len(path)-1]
	}
	if node.Right != nil {
		path = append(path, node.Right.Val)
		dfs(node.Right, targetSum, ans, path, sum+node.Right.Val)
		path = path[:len(path)-1]
	}
}
```

