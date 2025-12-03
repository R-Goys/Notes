[426. 将二叉搜索树转化为排序的双向链表](https://leetcode.cn/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/)

[LCR 155. 将二叉搜索树转化为排序的双向链表](https://leetcode.cn/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

> 将一个 **二叉搜索树** 就地转化为一个 **已排序的双向循环链表** 。
>
> 对于双向循环列表，你可以将左右孩子指针作为双向循环链表的前驱和后继指针，第一个节点的前驱是最后一个节点，最后一个节点的后继是第一个节点。
>
> 特别地，我们希望可以 **原地** 完成转换操作。当转化完成以后，树中节点的左指针需要指向前驱，树中节点的右指针需要指向后继。还需要返回链表中最小元素的指针。

----

原本以为有点难，但是做起来还行吧。

就是有一个坑，函数调用竟然是传的指针的副本，以前一直没有注意到这个。😭

总的思路是dfs，需要一个虚拟头节点维护整个链表，按照中序遍历的原则，将最先拿到的节点前驱节点指向链表的尾节点，链表的尾节点的后驱节点指向当前拿到的节点，这样持续下去，过程中，需要传递一个指针来维护这样一个尾节点。

```go
func treeToDoublyList(root *Node) *Node {
    if root == nil {
        return nil
    }
    dummy := &Node{}
    tail := dummy

    dfs(root, &tail)

    First := dummy.Right
    tail.Right = First
    First.Left = tail

    return dummy.Right
}

func dfs(root *Node, tail **Node) {
    if root == nil {
        return
    }
    dfs(root.Left, tail)
    (*tail).Right = root
    root.Left = (*tail)
    (*tail) = (*tail).Right
    dfs(root.Right, tail)
}
```

