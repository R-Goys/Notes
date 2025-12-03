[94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/)

> 给定一个二叉树的根节点 `root` ，返回 *它的 **中序** 遍历* 。

----

### 递归

```go
func inorderTraversal(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    var ans []int
    ans = append(ans, inorderTraversal(root.Left)...)
    ans = append(ans, root.Val)
    ans = append(ans, inorderTraversal(root.Right)...)
    return ans
}

```

-----

### 非递归

说一下思路：

递归的方法就是先将左侧的节点加入数组，再将中间的节点加入，最后是右边，既然如此，非递归也能模拟成递归，这里我不禁想起`程序就是状态机`这句话，一切程序都可以变成非递归，感到困惑不妨思考一下我们程序运行的时候的栈。

回到正题，程序运行不断将遇见函数的时候就会将函数压到栈中执行，反过来，我们去模拟栈又如何呢？

不断将右节点压入栈中，直至遇到nil，开始将我们的节点的Val加入答案数组，并出栈，再将我们的右节点加入栈中

```go
func inorderTraversal(root *TreeNode) []int {
    var st []*TreeNode
    var ans []int
    
    for root != nil || len(st) > 0 {
        for root != nil {
            st = append(st, root)
            root = root.Left
        }
        
        root = st[len(st)-1]
        st = st[:len(st)-1]
        ans = append(ans, root.Val) 
        
        root = root.Right 
    }
    
    return ans
}

```

