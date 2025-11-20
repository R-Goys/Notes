[21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

#  前言

由于以前一直写的数组模拟的链表，对于结构体链表的顾及比较少，偶然会想起结构体链表还不熟悉，故写此为，用来复习。

# 正文

题目是简单的有序地合并两个有序链表

因此我们要先创建一个新的链表来储存合并的结果

注意，这里如果要创建新的链表，首先要创建其头节点，此处需要new关键字来开辟新的空间

然后呢，则需要再创建一个ListNode的指针，来指向结果链表的最后一个节点，以达到向后一次按大小依次加入还未加入结果链表的节点的值，

除此之外，我们还需要两个结构体指针来指向当前比较数值的两个节点的指针

加入结果链表时，还需要new关键字来为结果链表的新开辟的节点开辟空间，加入之后，将新加入结果链表的节点的指针向后移动，即指向下一个节点，随后继续比较，直到遇到nullptr则结束比较。下面是代码的实现：

```cpp
class Solution {
public:
    // 合并两个有序链表并返回合并后的链表头指针
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode* Head = new ListNode(); // 创建一个虚拟头节点，方便处理合并链表
        ListNode* listres = Head; // listres 用于指向合并链表的当前节点

        ListNode* i = list1; // 指向 list1 的当前节点
        ListNode* j = list2; // 指向 list2 的当前节点

        // 当两个链表都有节点时，进行比较和合并
        while(i != nullptr && j != nullptr) {
            // 如果 list1 的当前节点值大于 list2 的当前节点值
            if(i->val > j->val) {
                listres->next = new ListNode(j->val); // 创建新的节点，并添加到合并链表中
                j = j->next; // 移动 list2 的指针到下一个节点
                listres = listres->next; // 移动指向合并链表的指针到下一个位置
                continue; // 继续进行下一个循环
            }
            // 如果 list1 的当前节点值小于 list2 的当前节点值
            if(i->val < j->val) {
                listres->next = new ListNode(i->val); // 创建新的节点，并添加到合并链表中
                i = i->next; // 移动 list1 的指针到下一个节点
                listres = listres->next; // 移动指向合并链表的指针到下一个位置
                continue; // 继续进行下一个循环
            }
        }

        // 当 list1 中还有剩余节点时，将它们添加到合并链表中
        while(i != nullptr) {
            listres->next = new ListNode(i->val); // 创建新的节点，并添加到合并链表中
            i = i->next; // 移动 list1 的指针到下一个节点
            listres = listres->next; // 移动指向合并链表的指针到下一个位置
        }

        // 当 list2 中还有剩余节点时，将它们添加到合并链表中
        while(j != nullptr) {
            listres->next = new ListNode(j->val); // 创建新的节点，并添加到合并链表中
            j = j->next; // 移动 list2 的指针到下一个节点
            listres = listres->next; // 移动指向合并链表的指针到下一个位置
        }

        // 返回合并后的链表，跳过虚拟头节点
        return Head->next;
    }
};
```

----

### 第二次写：

基本没啥问题....

```go
func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    tmp1 := list1
    tmp2 := list2
    dummy := &ListNode{}
    Tmp := dummy

    for tmp1 != nil && tmp2 != nil {
        if tmp1.Val <= tmp2.Val {
            Tmp.Next = tmp1
            Tmp = Tmp.Next
            tmp1 = tmp1.Next
        } else {
            Tmp.Next = tmp2
            Tmp = Tmp.Next
            tmp2 = tmp2.Next
        }
    }
    if tmp1 == nil {
        Tmp.Next = tmp2
    } else {
        Tmp.Next = tmp1
    }
    return dummy.Next
}
```

# 三刷

优雅一点的cpp

```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode* node1 = list1;
        ListNode* node2 = list2;
        ListNode* dummy = new ListNode();
        ListNode* tmp = dummy;

        while (node1 || node2) {
            if (node1 && node2) {
                if (node1->val < node2->val) {
                    tmp->next = node1;
                    node1 = node1->next;
                } else {
                    tmp->next = node2;
                    node2 = node2->next;
                }
                tmp = tmp->next;
                continue;
            }
            tmp->next = node1 ? node1 : node2;
            node1 = nullptr;
            node2 = nullptr;
        }
        return dummy->next;
    }
};
```





# 结语

简单的链表操作，可以用来复习关于结构体链表的知识，希望对你有帮助。



# ----------END------------