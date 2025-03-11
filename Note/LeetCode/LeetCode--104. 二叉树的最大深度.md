[104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

> 给定一个二叉树 `root` ，返回其最大深度。
>
> 二叉树的 **最大深度** 是指从根节点到最远叶子节点的最长路径上的节点数。

---

## 递归

递归太简单了，就是送的

```go
func maxDepth(root *TreeNode) int {
	if root == nil {
		return 0
	}
	return max(maxDepth(root.Left), maxDepth(root.Right)) + 1
}

```

## 广搜

由于是一层一层搜的，所以每搜一层，深度+1

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    ans := 0
    q := []*TreeNode{}
    q = append(q, root)
    for len(q) != 0 {
        sz := len(q)
        for sz > 0 {
            node := q[0]
            q = q[1:]
            if node.Left != nil {
                q = append(q, node.Left)
            }
            if node.Right != nil {
                q = append(q, node.Right)
            }
            sz--
        }
        ans ++
    }
    return ans
}
```

