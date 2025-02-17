[23. 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)

>给你一个链表数组，每个链表都已经按升序排列。
>
>请你将所有链表合并到一个升序链表中，返回合并后的链表。

----

## 正文

这道题有多种解决方案

### 堆

比较容易，又比较直观的就是堆排序，将每个节点加入最小根堆中，依次弹出加入最后的链表，就可得出答案，事实上，并不需要每次都将所有链表加入，只需要最开始将每个链表的头节点加入，然后在弹出链表时，直接将弹出的节点的下一个节点再加入堆即可，这样能够有效节省空间。

代码如下：

```go
func mergeKLists(lists []*ListNode) *ListNode {
    lh := &ListHeap{}
    heap.Init(lh)
    for _, node := range lists {
        if node != nil {
            heap.Push(lh, node)
        }
    }
    dummy := &ListNode{}
    tmp := dummy
    for lh.Len() > 0 {
        Node := heap.Pop(lh).(*ListNode)
        tmp.Next = Node
        tmp = tmp.Next
        if Node.Next != nil {
            heap.Push(lh, Node.Next)
        }
    }
    return dummy.Next
}

type ListHeap []*ListNode

func (l *ListHeap) Len() int {
	return len(*l)
}

func (l *ListHeap) Less(i, j int) bool {
	return (*l)[i].Val < (*l)[j].Val
}

func (l *ListHeap) Swap(i, j int) {
	(*l)[i], (*l)[j] = (*l)[j], (*l)[i]
}

func (l *ListHeap) Push(x any) {
	*l = append(*l, x.(*ListNode))
}

func (l *ListHeap) Pop() any {
	res := (*l)[len(*l)-1]
	*l = (*l)[:len(*l)-1]
	return res
}

```

堆排序不用ide也太难写了~

### 分治

跟归并排序的思路类似，将链表切片分成两部分，分别合并成一个链表，再将这两个链表进行合并。

可以理解为：

```
	链表1  链表2  链表3  链表4
	|		 |		|		|
	|		 |		|		|
	|		 |		|		|
	|		 |		|		|
	+——-+-——+     +———+——+
		 |			 	 |
		 链表			  链表
		 +------+——————+
          		|
				  |
				最终链表
```



代码如下：

```go
func mergeKLists(lists []*ListNode) *ListNode {
    return Merge(lists, 0, len(lists) - 1)
}

func Merge(lists []*ListNode, l int, r int) *ListNode {
    if l == r {
        return lists[l]
    } else if l > r {
        return nil
    }
    mid := (l + r) / 2
    return MergeTwoLists(Merge(lists, l, mid), Merge(lists, mid + 1, r))
}

func MergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    dummy := &ListNode{}
    tmp := dummy
    for list1 != nil && list2 != nil {
        if list1.Val > list2.Val {
            tmp.Next = list2
            tmp = tmp.Next
            list2 = list2.Next
        } else {
            tmp.Next = list1
            tmp = tmp.Next
            list1 = list1.Next
        }
    }
    if list1 != nil {
        tmp.Next = list1
    }
    if list2 != nil {
        tmp.Next = list2
    }
    return dummy.Next
}
```

---

