[236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)

> 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
>
> [百度百科](https://baike.baidu.com/item/最近公共祖先/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

----

## 正文

根据递归的性质，在同一路径上，先遍历到的节点一定是后遍历到的节点的祖宗。

最近公共祖先有两种情况：

1. 其中一个节点为最近公共祖先
2. 两个节点所在的路径相交的那个节点为最近公共节点。

下面为代码，由于光是文字不好表达，我在代码块中嵌入注释来解释思路

```go
 func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
     //找到了或者当前寻找的路径为空，直接返回当前节点，帮助判断
    if root == nil || p == root || q == root {
        return root
    }
     //计算得出当前root的左儿子中是否存在p或者q，如果不存在则返回nil
     //如果存在，则返回p或q
    l := lowestCommonAncestor(root.Left, p, q)
     //同上
    r := lowestCommonAncestor(root.Right, p, q)
     //如果左右儿子都找到了，说明当前节点root就是最近公共祖先
    if l != nil && r != nil {
        return root
    }
     //说明并非左右儿子都找到了p或q，说明p或q其中一个是最近公共祖先
    if l != nil {
        return l
    }
    return r
     //递归传递l或者r，返回最终的答案。
}
```

