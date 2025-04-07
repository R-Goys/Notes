[622. 设计循环队列](https://leetcode.cn/problems/design-circular-queue/)

> 设计你的循环队列实现。 循环队列是一种线性数据结构，其操作表现基于 FIFO（先进先出）原则并且队尾被连接在队首之后以形成一个循环。它也被称为“环形缓冲器”。
>
> 循环队列的一个好处是我们可以利用这个队列之前用过的空间。在一个普通队列里，一旦一个队列满了，我们就不能插入下一个元素，即使在队列前面仍有空间。但是使用循环队列，我们能使用这些空间去存储新的值。
>
> 你的实现应该支持如下操作：
>
> - `MyCircularQueue(k)`: 构造器，设置队列长度为 k 。
> - `Front`: 从队首获取元素。如果队列为空，返回 -1 。
> - `Rear`: 获取队尾元素。如果队列为空，返回 -1 。
> - `enQueue(value)`: 向循环队列插入一个元素。如果成功插入则返回真。
> - `deQueue()`: 从循环队列中删除一个元素。如果成功删除则返回真。
> - `isEmpty()`: 检查循环队列是否为空。
> - `isFull()`: 检查循环队列是否已满。

---

这是一个队列，需要我们去维护队头指针和队尾指针，个人的解法不太清晰，就是一坨屎山，而官方的题解给出了取模来减少了多余的判断，所以我觉得官方的更好一点，循环队列的下标可以用取模来跟踪！这是我学到的。

```go
type MyCircularQueue struct {
    front, rear int
    elements    []int
}

func Constructor(k int) MyCircularQueue {
    return MyCircularQueue{
        //为什么是k + 1？
        //留下一个空的存储单元，便于判断队列已满和队列为空。
        elements: make([]int, k + 1),
    }
}

func (q *MyCircularQueue) EnQueue(value int) bool {
    if q.IsFull() {
        return false
    }
    q.elements[q.rear] = value
    q.rear = (q.rear + 1) % len(q.elements)
    return true
}


func (q *MyCircularQueue) DeQueue() bool {
    if q.IsEmpty() {
        return false
    }
    q.front = (q.front + 1) % len(q.elements)
    return true
}


func (q *MyCircularQueue) Front() int {
    if q.IsEmpty() {
        return -1
    }
    return q.elements[q.front]
}


func (q *MyCircularQueue) Rear() int {
    if q.IsEmpty() {
        return -1
    }
    return q.elements[(q.rear - 1 + len(q.elements)) % len(q.elements)]
}


func (q *MyCircularQueue) IsEmpty() bool {
    return q.rear == q.front
}


func (q *MyCircularQueue) IsFull() bool {
    return (q.rear + 1) % len(q.elements) == q.front
}
```

