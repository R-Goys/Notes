[232. 用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/)

> 请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：
>
> 实现 `MyQueue` 类：
>
> - `void push(int x)` 将元素 x 推到队列的末尾
> - `int pop()` 从队列的开头移除并返回元素
> - `int peek()` 返回队列开头的元素
> - `boolean empty()` 如果队列为空，返回 `true` ；否则，返回 `false`
>
> **说明：**
>
> - 你 **只能** 使用标准的栈操作 —— 也就是只有 `push to top`, `peek/pop from top`, `size`, 和 `is empty` 操作是合法的。
> - 你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。

---

这题感觉有点内啥，但是还是有一些值得思考的地方，比如说要弹出栈中元素的时候，只有在当前out切片为空的时候才去填充，这个思路很好。

```go
type MyQueue struct {
    In, Out []int
}


func Constructor() MyQueue {
    return MyQueue{}
}


func (this *MyQueue) Push(x int)  {
    this.In = append(this.In, x)
}


func (this *MyQueue) Pop() int {
    if len(this.Out) == 0 {
        this.InToOut()
    }
    x := this.Out[len(this.Out) - 1]
    this.Out = this.Out[:len(this.Out) - 1]
    return x
}


func (this *MyQueue) Peek() int {
    if len(this.Out) == 0 {
        this.InToOut()
    }
    return this.Out[len(this.Out) - 1]
}


func (this *MyQueue) Empty() bool {
    return len(this.Out) == 0 && len(this.In) == 0
}

func (this *MyQueue)InToOut() {
    for len(this.In) > 0 {
        this.Out = append(this.Out, this.In[len(this.In) - 1])
        this.In = this.In[:len(this.In) - 1]
    }
}

```

