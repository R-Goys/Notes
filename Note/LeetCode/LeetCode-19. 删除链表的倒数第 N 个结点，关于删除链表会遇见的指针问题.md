[19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

#  前言

虽然这道题比较简单，但是我在做这道题时发现了几个需要注意的地方，故写个笔记提一下。

# 正文

先将源代码给出来

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* x = new ListNode();
        x->next = head;
        ListNode* p = x;
        ListNode* p2 = x;
        int idx = 0;
        do {
            idx++;
            p = p->next;
        } while (p != nullptr);

        for (int i = 1; i < idx - n; i++) {
            p2 = p2->next;
        }
        ListNode* TMP = p2->next;
        p2->next = TMP->next;
        delete TMP;
        return x->next;
    }
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

思路很简单，遍历找到链表的长度，然后再次遍历找到链表应该删除的节点，随后将其删除，但是有人(比如我)可能会犯一个错误,在答案的最后直接返回head，然而事实上在删除的节点是第一个节点的时候，这种做法是会出错的，

关于这一点，涉及我们对指针的理解。

下面来分析一下：

众所周知，链表的删除操作是依靠修改节点的 `->next` 指针。在这个过程中，我们需要创建一个虚拟节点以便找到要删除的节点。当我们找到需要删除的节点时，节点的前一个节点是 `p2`，而要删除的节点是 `TMP`。

当需要删除的节点不是第一个节点时，删除操作能够正确进行。这是因为我们创建的 `p2` 指针是一个新的指针，仍然指向链表的虚拟节点，而这个虚拟节点的 `->next` 指向链表的头节点。当我们执行 `p2->next = TMP->next` 这一操作时，实际上是将 `p2` 所指向的虚拟节点的 `next` 指针指向了要删除节点的下一个节点，这样就能够修改链表本身的结构，从而成功地完成删除操作。

然而，当我们需要删除第一个节点时，节点的前一个节点就是虚拟节点。此时修改虚拟节点的 `->next` 指针将使其指向链表的第二个节点。然而这个虚拟节点事实上并不存在于链表中，仅仅是修改这个虚拟节点的数值或是->next指针并不能达成修改链表的效果，因为虚拟节点的->next只是指向了head节点，如果修改这个->next值，仅仅只能表示指向改变了，而不是head改变了，所以修改这个->next指针并不能实现在head指向的链表中的第一个节点被删除的结果。

因此，通过修改虚拟节点的 `->next` 指针，我们实际上可以删除链表中的第一个节点，并确保链表的结构得到正确更新。因此，我们应当返回虚拟节点的 `->next` 指针，而不是直接返回 `head`。

# 结语

以上的想法是我自己思考的，如果有不对的地方麻烦请指出来~