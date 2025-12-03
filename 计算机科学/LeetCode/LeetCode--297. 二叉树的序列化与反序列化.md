[297. 二叉树的序列化与反序列化](https://leetcode.cn/problems/serialize-and-deserialize-binary-tree/)

> 序列化是将一个数据结构或者对象转换为连续的比特位的操作，进而可以将转换后的数据存储在一个文件或者内存中，同时也可以通过网络传输到另一个计算机环境，采取相反方式重构得到原数据。
>
> 请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。
>
> **提示:** 输入输出格式与 LeetCode 目前使用的方式一致，详情请参阅 [LeetCode 序列化二叉树的格式](https://support.leetcode.cn/hc/kb/article/1567641/)。你并非必须采取这种方式，你也可以采用其他的方法解决这个问题。

---

另外有几种遍历方式也可以构造一个二叉树，不过平时做题已经习惯了先序遍历了(力扣给的二叉树样例也是这种格式)。

```go
type Codec struct {
    
}

func Constructor() Codec {
    return Codec{}
}

// Serializes a tree to a single string.
func (this *Codec) serialize(root *TreeNode) string {
    strb := &strings.Builder{}
    var dfs func(*TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            strb.WriteString("null,")
            return
        }
        strb.WriteString(strconv.Itoa(node.Val))
        strb.WriteByte(',')
        dfs(node.Left)
        dfs(node.Right)
    }
    dfs(root)
    return strb.String()
}

// Deserializes your encoded data to tree.
func (this *Codec) deserialize(data string) *TreeNode {    
    sp := strings.Split(data, ",")
    var build func() *TreeNode
    build = func() *TreeNode {
        if sp[0] == "null" {
            sp = sp[1:]
            return nil
        }
        val, _ := strconv.Atoi(sp[0])
        sp = sp[1:]
        return &TreeNode{Val:val, Left:build(), Right:build()}
    }
    return build()
}
```

