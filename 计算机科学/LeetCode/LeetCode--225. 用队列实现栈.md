[225. 用队列实现栈](https://leetcode.cn/problems/implement-stack-using-queues/)

> 请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（`push`、`top`、`pop` 和 `empty`）。
>
> 实现 `MyStack` 类：
>
> - `void push(int x)` 将元素 x 压入栈顶。
> - `int pop()` 移除并返回栈顶元素。
> - `int top()` 返回栈顶元素。
> - `boolean empty()` 如果栈是空的，返回 `true` ；否则，返回 `false` 。
>
>  
>
> **注意：**
>
> - 你只能使用队列的标准操作 —— 也就是 `push to back`、`peek/pop from front`、`size` 和 `is empty` 这些操作。
> - 你所使用的语言也许不支持队列。 你可以使用 list （列表）或者 deque（双端队列）来模拟一个队列 , 只要是标准的队列操作即可。

---

栈实现队列反过来，也是采取双栈思路，真正的栈放在其中一个队列，另一个栈用于在入栈的时候辅助MyStack能够正确入栈。

```go
type MyStack struct {
    q1 []int
    q2 []int
}

func Constructor() MyStack {
    return MyStack{}
}

func (s *MyStack) Push(x int)  {
    s.q1 = append(s.q1, x)
    for len(s.q2) > 0 {
        s.q1 = append(s.q1, s.q2[0])
        s.q2 = s.q2[1:]
    }
    s.q1, s.q2 = s.q2, s.q1
}

func (s *MyStack) Pop() int {
    num := s.q2[0]
    s.q2 = s.q2[1:]
    return num
}

func (s *MyStack) Top() int {
    return s.q2[0]
}

func (s *MyStack) Empty() bool {
    return len(s.q2) == 0
}
```



