[155. 最小栈](https://leetcode.cn/problems/min-stack/)

> 设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。
>
> 实现 `MinStack` 类:
>
> - `MinStack()` 初始化堆栈对象。
> - `void push(int val)` 将元素val推入堆栈。
> - `void pop()` 删除堆栈顶部的元素。
> - `int top()` 获取堆栈顶部的元素。
> - `int getMin()` 获取堆栈中的最小元素。

----

两种写法，辅助栈的思想也很不错，不过Pair结构体有浑然一体的感觉，我觉得更好。

### Pair

```go
type pair struct {
    val int
    preMin int
}

type MinStack []pair


func Constructor() MinStack {
    return MinStack {{0, math.MaxInt}}
}


func (this *MinStack) Push(val int)  {
    *this = append(*this, pair{val, min(this.GetMin(), val)})
}


func (this *MinStack) Pop()  {
    *this = (*this)[:len(*this) - 1]
}


func (this *MinStack) Top() int {
    return (*this)[len(*this) - 1].val
}


func (this *MinStack) GetMin() int {
    return (*this)[len(*this) - 1].preMin
}

```

### 辅助栈

```go
type MinStack struct {
    stack []int
    minStack []int
}


func Constructor() MinStack {
    return MinStack{
        stack: []int{},
        minStack: []int{math.MaxInt64},
    }
}


func (this *MinStack) Push(val int)  {
    this.stack = append(this.stack, val)
    top := this.minStack[len(this.minStack) - 1]
    this.minStack = append(this.minStack, min(top, val))
}


func (this *MinStack) Pop()  {
    this.stack = this.stack[:len(this.stack) - 1]
    this.minStack = this.minStack[:len(this.minStack) - 1]
}


func (this *MinStack) Top() int {
    return this.stack[len(this.stack) - 1]
}


func (this *MinStack) GetMin() int {
    return this.minStack[len(this.minStack) - 1]
}

```

cpp 写法

```cpp
class MinStack {

private:
    stack<int> stk;
    stack<int> minstk;
public:
    MinStack() {
        this->minstk.push(INT_MAX);
    }
    
    void push(int val) {
        this->stk.push(val);
        this->minstk.push(min(val, this->getMin()));
    }
    
    void pop() {
        this->stk.pop();
        this->minstk.pop();
    }
    
    int top() {
        return this->stk.top();
    }
    
    int getMin() {
        return this->minstk.top();
    }
};

```

