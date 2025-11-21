# 关于Golang的一些碎片知识

因为平时看的时间比较杂，没有系统的学Golang，最近在看Go语言设计与实现，感觉一些有趣，有难度的知识点就在这里用自己的话总结一下，写下来了，主要还是供自己复习用的，有些我觉得比较清晰的地方就没有写太多，感觉很模糊可以去看原书，讲得很透彻。



----

## 1. GMP模型

### GMP模型及运行原理

G代表goroutine，M代表machine也叫做线程，P代表processor，处理器(不是CPU！！)，G在线程上执行，由P进行调度。

这里大概讲一下调度器的运行过程，其中的很多细节都值得去仔细看，如果有需要，可以到《[Go 语言设计与实现](https://draveness.me/golang)》上阅读：

1. 在最开始，我们的调度器启动，会根据`GOMAXPROCS`来更新P的数量，一个P绑定一个M，同一时刻每个MP只能执行一个G，多余的G，其状态为_Grunnable形成队列在后面等待，也就是说，一个线程可以管理多个协程！
2. 创建goroutine，使用`go [ 函数 ]`的方式来创建一个goroutine，当然如果之前已经创建过goroutine，并且那个goroutine已经变成空闲状态的话，就会从这些空闲状态的goroutine取一个，也就是复用goroutine，而不是创建，执行完这一步之后，会将当前goroutine加入到运行队列。(tips：其实这部分的源码很有趣，还不熟悉gmp调度的同学可以先忽略括号的内容，通过go关键字来获取一个新的G的时候，会先获取当前正在运行的G，同时通过这个G拿到P，然后看这个P维护的goroutine队列是否存在空闲的goroutine，如果不存在，就再去全局的goroutine队列里面找，如果还是不存在，那就会自己创建一个新的goroutine，最后再将栈指针，程序计数器等参数保存到当前goroutine中，最后更新状态然后将其推送到运行队列中)
3. 注意，刚加入运行队列的时候，这个goroutine还处于睡眠状态，没有真正运行，当满足条件的时候，他会被唤醒，然后到队首执行。
4. **接下来要介绍一下调度循环，也就是真正的goroutine执行了**
5. 调度器启动之后，会先初始化**G0**(这是一个特殊的goroutine，创建新 Goroutine、执行栈切换、执行垃圾回收，都有它的身影)，然后再进入调度循环。
6. 调度循环中，每次调度都会从全局和本地的运行队列中查找goroutine，如果都没找到，就会进行阻塞性的查找：从本地，全局运行队列中找，从网络轮询器中找，或者窃取待运行的goroutine，总之，这里一定会返回一个goroutine。
7. 找到之后，就会将其调度到线程M上(此时，注意了，GMP轻量的关键之一体现在这里，他将goroutine结构体中保存的相关数据恢复到cpu的寄存器上，从而实现了用户级调度，而不会触发系统调用)
8. 在当前goroutine的执行已经结束的时候，又会清除其中的字段，转换成_GDead状态，并且加入处理器的空闲列表中，然后切换到下一个待执行的goroutine。

### 调度

1. **主动挂起**：调用[`runtime.gopark`](https://draveness.me/golang/tree/runtime.gopark)令当前goroutine暂停，并且移除运行队列，进入休眠，当满足一定条件，会再度唤醒，并且被加入运行队列
2. **系统调用**：go语言封装了系统调用，使得在执行系统调用的时候，Golang能够做出相应的准备和清理(保存和恢复goroutine以及其他的处理)
3. **协作式**：可以理解为“我休息一会，你们先上”，就是让出处理器，将自己状态变成_Grunnable，放到全局队列，然后重新开始调度；另一种是函数调用时，会在前方插入一个触发抢占的函数，检查是否有发出抢占请求的goroutine。
4. **基于信号的抢占式调度**：通过 `SIGURG` 信号和修改寄存器，强制让出 CPU，解决紧密循环 Goroutine 无法被调度的问题，提高调度公平性和 GC 效率。



----

## 2. Context上下文

context不是很形象，就和他的名字一样,**上下文**，它可以根据上下文的参数传递，函数调用等来执行操作，和goroutine联系紧密。

### **最佳实践**

-  合理利用 `context.WithCancel` 取消 Goroutine，避免泄露，减少资源占用
-  使用 `context.WithTimeout` 限制任务时间，防止超时阻塞
-  使用 `context.WithDeadline` 确保任务在固定时间前完成
-  使用 `context.WithValue` 传递请求作用域数据(这个不常用)
-  `context` 只应该作为参数传递，而不是结构体字段，更不是全局变量



----

## 3. 计时器

表面上是切片，内部使用四叉堆维护timer(计时器)，堆顶元素是距离截止时间最近的计时器，四叉堆，顾名思义，就是每个节点只能有四个子节点，Go语言通过系统监控和调度器来检查timer是否到期，如果到期，则会执行回调。

### **最佳实践**

- 使用 `time.Timer` 或 `time.Ticker` 进行定时任务

   ```go
   timer := time.NewTimer(2 * time.Second)
   <-timer.C // 2秒后触发
   ```

2. 避免使用 `time.Sleep` 影响调度

   ```go
   select {
   case <-time.After(2 * time.Second):
       fmt.Println("2秒后执行")
   }
   ```

   这样不会阻塞当前 Goroutine，适用于需要超时控制的场景。

3. 使用 `time.AfterFunc` 注册回调

   ```go
   time.AfterFunc(1*time.Second, func() {
       fmt.Println("1秒后触发")
   })
   ```

   适用于触发一次的定时任务。

4. 使用 `Ticker` 处理周期性任务

   ```go
   ticker := time.NewTicker(1 * time.Second)
   defer ticker.Stop()
   for range ticker.C {
       fmt.Println("每秒执行一次")
   }
   ```

   `Ticker` 适用于高效的循环任务，避免每次创建新的 `Timer`。

5. 及时停止 `Timer` 和 `Ticker` 以释放资源

   ```go
   timer := time.NewTimer(5 * time.Second)
   timer.Stop() // 停止计时器，防止 Goroutine 泄漏
   ```

6. 注意 `select` 防止 Goroutine 泄漏(timer.After内部会创建一个goroutine)

   ```go
   select {//错误做法
   case <-time.After(3 * time.Second):
       fmt.Println("超时")
   case result := <-someChannel:
       fmt.Println("收到结果", result)
   }
   ```

7. 避免 `time.After` 造成的 Goroutine 泄漏(timer.After内部会创建一个goroutine)

   ```go
   timer := time.NewTimer(5 * time.Second)
   defer timer.Stop()
   select {//true做法
   case <-timer.C:
       fmt.Println("超时")
   case <-done:
       fmt.Println("任务完成")
   }
   ```

**tips**：

我在看原书的时候这两部分直接让我大脑宕机了：

>1. 因为目前的计时器由网络轮询器管理和触发，它能够让网络轮询器立刻返回并让运行时检查是否有需要触发的计时器。						 			   --《网络轮询器》部分
>2. 这里将分析器的触发过程，Go 语言会在两个模块触发计时器，运行计时器中保存的函数：
>   - 调度器调度时会检查处理器中的计时器是否准备就绪；
>   - 系统监控会检查是否有未执行的到期计时器；		--《定时器》部分

最开始让我感觉很矛盾，事实上，我认为这些触发是相辅相成的，调度器作为主导的触发计时器的组件，而系统监控作为辅助，网络轮询器作为辅助，当网络轮询器陷入`poll_wait`状态时，当前M线程陷入休眠，此时这个M陷入系统调用，不会去执行其他的goroutine，与其让这个M睡死，不如直接唤醒它，让他来执行timer，也可以避免另外两种触发timer的方法会创建新的M造成的资源浪费。

---

## 4. 网络轮询器

重写一下，回来一看之前的理解有很大的错误。

Go 的网络轮询器其实就是为网络 I/O 做了一层优化，深入到 `net/http` 的源码我们可以发现，他就是基于 pollDesc 进行实现的监听操作，而 go 的网络轮询器在 linux 上的实现就是利用 epoll 的多路复用来监听网络 I/O，然而，其实这和我们自己实现一个 reactor 模式的网络库并没有本质区别，go 的网络轮询器就是用一个后台的协程去帮你监听而已，`net/http` 和 `hertz` 的底层并不一样，`hertz` 的高性能并没有去依赖 Go 中内置的网络轮询器，而是自己实现了一个名叫 `netpoll` 的网络库，其实就是一个 reactor 模式 + 协程池 + buffer 复用。

然而，Go 的网络轮询器到目前为止也只对网络 I/O 有作用，针对于文件 I/O 并没有效果，期待有朝一日能将 io_uring 加入 Go Runtime，我个人没什么研究，但是如果能将 io_uring 加入 go runtime 的话，至少对于文件 I/O 会有极大的性能提升，至于网络轮询器我感觉这样继续采用 epoll 就好。

----

## 5. Channel

Go中的 Channel 收发操作均遵循了先进先出的设计：

- 先从 Channel 读取数据的 Goroutine 会先接收到数据。
- 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利。

虽然目前go希望能够实现无锁的channel来提高性能，但是目前底层的channel依旧使用锁(Lock)来维护channel队列先进先出的特性。

### CSP模型

> - 模块化设计，每个模块专注于一个任务
> - 避免共享内存，采取通信的方式(消息传递)进行通信
> - 不同对象相互隔离，通过**管道**通信

### 数据结构

底层的数据结构中，有几个字段值得一提：

- `buf` — Channel 的缓冲区数据指针；
- `sendx` — Channel 的发送操作处理到的位置；
- `recvx` — Channel 的接收操作处理到的位置；
- `dataqsiz` - Channel的缓冲区大小

这几个字段，维护了Channel的信息接受与发出，buf指向缓冲区的首地址，sendx指向接下来要如果执行ch <- msg时，向管道发出信息存储的位置，随后sendx++，移动到下一个位置，recvx则指向接下来会将要被发出的信息的位置，然后recvx++，也就是移动到下一个位置。

看起来貌似sendx和recvx会移动？别担心，当sendx或者recvx超出了这个缓冲区的范围，也就是超出了dataqsiz的大小，此时会归0，也就是移动到第一个字段，下面有个图帮助理解：

```go
初始状态:
[ _, _, _ ]   sendx=0, recvx=0

ch <- 1
[ 1, _, _ ]   sendx=1, recvx=0

ch <- 2
[ 1, 2, _ ]   sendx=2, recvx=0

ch <- 3
[ 1, 2, 3 ]   sendx=3, recvx=0  (sendx == 3, 下一次会回绕)

<- ch
[ _, 2, 3 ]   sendx=3, recvx=1

<- ch
[ _, _, 3 ]   sendx=3, recvx=2

ch <- 4
[ 4, _, 3 ]   sendx=0 (回绕), recvx=2

ch <- 5
[ 4, 5, 3 ]   sendx=1, recvx=2
```

这样就是我们的一个**循环队列**了。

**tips**：

当队列已满，我们需要向其中发送信息怎么办？此时会将当前goroutine陷入休眠，此时如果没有其他goroutine接受channel中的数据，此时也就形成了死锁。

当然，如果当前channel为空，而我们要从中那个接受消息，当前goroutine也会陷入休眠，直到有其他goroutine向channel中发送了信息。

**Q**：如果有多个goroutine发送或接受信息，是怎么排队的？**A**：底层的Channel数据结构中还有一个成员`recvq`和`sendq`其中存储了goroutine的队列，以此来确保先后顺序，其中还有一个`sudog`的结构体对象，用来真正存储goroutine的链表和相关信息。

当发生发生这种情况时，如果关闭这个channel，就会解决死锁的问题，但是如果时向已经满的channel发送数据造成的死锁，那么会引发panic。

### 非阻塞channel

以上是阻塞式的channel，但是如果想要实现非阻塞的怎么办？很简单，和select一起使用就行，举个例子：

```go
select {
case ch <- value:
    fmt.Println("发送成功")
case val := <-ch:
    fmt.Println("收到数据:", val)
default:
    fmt.Println("无操作，通道不可读也不可写")
}
```

### 最佳实践

| **最佳实践**                                         | **示例**                     | **适用场景**                |
| ---------------------------------------------------- | ---------------------------- | --------------------------- |
| **非阻塞 `channel`**                                 | `select { case <-ch: }`      | 轮询、日志收集              |
| **正确关闭 `channel`**                               | `close(ch)` + `for range ch` | 生产者-消费者模型           |
| **使用 `sync.WaitGroup`**                            | `wg.Add(1)` + `wg.Wait()`    | 等待所有 Goroutine 完成     |
| **使用缓冲 `channel`**                               | `make(chan int, 3)`          | 提高吞吐量                  |
| **避免 `nil channel`**                               | `ch := make(chan int)`       | 确保 `channel` 被正确初始化 |
| **使用 `context` 控制退出而不是**`channel`           | `context.WithCancel`         | 任务取消、超时控制          |
| **使用 `fan-in` 和 `fan-out`**(不知道可以自己搜一下) | `merge()` / `worker()`       | 并发处理任务                |

---

## 6. for和range

1. 如果使用for-range遍历切片，并不断在后面追加元素，并不会造成无限循环，只会遍历到切片最初的长度，而如果使用`for i := 0; i < len(slice); i ++`的时候，则会陷入无限循环。
2. 如果使用`for i, v := range slice`，此时的i和v均为临时变量，如果将地址v的地址加入一个新的数组，则会出问题。
3. for range遍历哈希表的时候，顺序随机。



----

## 7. select

select，与switch类似，但是select必须与channel搭配使用，他会在满足条件的case中随机选择一个进行执行，和select搭配，channel就可以实现非阻塞的收发数据。

值得一提的是，如果不存在满足条件的case，就会阻塞下去，这时候可以加一个default字段，表示所有的case都不满足条件的case，此时select中的channel会**非阻塞的执行**

### 数据结构

select不存在相应的结构体，但是case存在对应的数据结构：

```go
type scase struct {
	c    *hchan         // chan
	elem unsafe.Pointer // data element
}
```

### 实现

1. 将所有的 `case` 转换成包含 Channel 以及类型等信息的 [`runtime.scase`](https://draveness.me/golang/tree/runtime.scase) 结构体；
2. 调用运行时函数 [`runtime.selectgo`](https://draveness.me/golang/tree/runtime.selectgo) 从多个准备就绪的 Channel 中选择一个可执行的 [`runtime.scase`](https://draveness.me/golang/tree/runtime.scase) 结构体；
3. 通过 `for` 循环生成一组 `if` 语句，在语句中判断自己是不是被选中的 `case`；

`runtime.selectgo`是如何实现的？让我们来分析分析：

首先最开始，我们要先确定这些case执行的顺序(随机确定)，然后决定每个通道加锁的顺序，随后再加锁完成之后就会进入selectgo函数的主循环：

1. 第一阶段是寻找已经就绪的case，如果case不包含channel，就会跳过，也就是不会接收到数据，如果当前case有channel的时候，就会尝试去拿数据，但是如果没有数据可以读，并且管道已经关闭了。如果需要向channel中发送数据，会先检查通道是否关闭，如果关闭，则会触发panic，否则，正常判断。如果遇到了default，表示之前的cas而都没有执行，所以会直接执行default事件。
2. 如果没有default，并且没有找到对应的已经就绪的case，那么就会进入到第二阶段，将当前的goroutine加入到各个管道的等待队列中进行等待，进入休眠状态。
3. 一旦其中的一些channel就绪了，就会唤醒goroutine，进入第三阶段。我们需要先从goroutine中拿取节点`sudog`来找到对应的就绪的channel，如果找到一个channel，剩下的case都不会被执行，但是由于其他的channel可能也是就绪状态，我们还需要将这些废弃的节点`sudog`从这些没有执行的channel里面删除(如果不知道`sudog`是什么，可以回头看看channel的内容)，但是此时也依旧是通过遍历去查找所有的case，比对，然后对选择的case的索引进行保存，其余的则删除。这个时候，我们就完成了我们的选择了。



----

## 8. 系统监控

**系统监控**，是Go语言内部一个随着程序的启动而启动的循环，也会随着程序的终止而终止，它独立运行在一个**线程**中，在循环的内部会轮询网络、抢占长期运行或者处于系统调用的 Goroutine 以及触发垃圾回收。

### 检查死锁

系统监控通过[`runtime.checkdead`](https://draveness.me/golang/tree/runtime.checkdead) 来检查是否出现了死锁，具体分为以下三步

1. 检查是否有正在运行的线程
2. 检查是否有正在运行的goroutine
3. 检查是否存在正在等待的timer

为什么是这三步应该很好想，这里就不作介绍了。

### 运行计时器

计时器实际上也可以由这个系统监控来触发。

怎么去处理计时器？系统监控会通过遍历所有P去寻找他们的timers的堆顶元素，并且返回最早到期的时间，如果没有需要触发的计时器，那么会陷入休眠，但是如果这个时间极短，会等待直到时间过期。除此之外，如果发现当前timer已经过期，这说明timer对应的P中所有goroutine都在忙碌，此时会启动一个新的M线程去执行这个timer，以避免计时器延迟过大。

### 轮询网络

如果上一次轮询网络已经过去了 10ms，那么系统监控还会在循环中轮询网络，检查是否有待执行的文件描述符，此时会**非阻塞**的调用netpoll(与之前讲的阻塞调用netpoll不一样)来检查是否有fd准备就绪，这里的系统监控也是一种轮询网络的方法。

### 抢占处理器

系统监控还会遍历全局的处理器，来检查防止某一个goroutine占用M的时间过长，以此防止饥饿问题。

### 垃圾回收

这个在垃圾回收的部分详细解释吧。



----

## 9. 内存分配器

大多数编程语言的内存分配主要有两种方法：**线性分配**和**空闲链表分配**，各有优缺点。Go 语言的内存管理结合了这两种方法的优点，并采用 **多级缓存策略** 来提高内存分配的效率。

### 设计

Go 语言的内存管理采用 **HeapArena** 作为基本单位，将一整块连续的大内存划分成多个 **HeapArena**，并通过多级结构进行管理，其层级如下：

> HeapArena → mspan → 偏移量

其中，`HeapArena` 是堆管理的基本单元，每个 `HeapArena` 会被进一步划分为多个 `mspan`，而 `mspan` 负责管理大小相近的对象。

### 内存分配的多级缓存

Go 语言的内存分配涉及多个缓存层级，以减少直接向操作系统申请内存的开销，提高分配效率。主要的缓存层级包括：

1. **线程缓存（mcache）**：每个线程（M）持有一个独立的 `mcache`，用于快速分配小对象，避免频繁加锁。
2. **中心缓存（mcentral）**：全局 136 个 `mcentral` 共享所有 `mcache`，当线程缓存中的内存不足时，会从 `mcentral` 请求新的内存块。但由于多个线程共享中心缓存，访问时需要加锁。
3. **页堆（heap）**：整个堆的核心管理结构，负责管理所有 `HeapArena`，当 `mcentral` 无法满足内存请求时，会从 `heap` 申请新的 `mspan`。如果 `heap` 也无法满足，则会向 **操作系统** 申请新的内存。

在 **中心缓存（mcentral）** 中，`mspan` 负责管理不同大小的对象，而在**页堆**级别，则使用 `HeapArena` 作为更大粒度的管理单元。当 `mcentral` 向 `heap` 申请内存时，实际上是将 `HeapArena` 进一步划分为 `mspan`。

此外，**线程缓存（mcache）** 还包含一个**微分配器**，通过**偏移量**来快速定位和分配微对象的内存

### 小对象与大对象的分配路径

- **小对象（<= 32KB）**：优先从 `mcache` 分配，如果 `mcache` 不足，则依次从 `mcentral` 和 `heap` 申请。最终如果 `heap` 也无法满足，则请求操作系统分配新的 `HeapArena`。
- **大对象（> 32KB）**：不会经过 `mcache` 或 `mcentral`，而是直接在 `heap` 上分配内存。

这样设计的好处是，在高并发场景下，绝大多数的小对象分配都可以在 `mcache` 层完成，避免了全局加锁，提高了内存分配的效率。



----

## 10. 垃圾收集器

### 三色标记法

#### 组成

**黑色**：活跃的对象，包括不存在引用任何外部指针或根节点可达的对象。

**灰色**：活跃的对象，被其他对象引用，同时也可能引用了其他白色的元素，所以需要对他进行遍历检查。

**白色**：可能为垃圾，所以需要遍历检查。

#### 工作原理

1. 从灰色对象中选一个变成黑色。
2. 将黑色对象指向的所有对象标记成灰色。
3. 循环上述过程

最终，所有引用和被引用的对象都变成了黑色，剩下的白色没有被任何对象引用，也没有引用其他对象，所以白色可以视为垃圾，应当被垃圾收集器(GC)回收。

到目前为止，一切正常，但是如果我们考虑到并发性，仅仅是这样的三色标记法是无法保证垃圾被正确回收的，比如我们来看一个例子，此时有A,B,C三个对象，A引用了B，C目前是垃圾：

> 1. 三色标记法执行，将A标记为黑色，B为灰色，C为白色。
> 2. 程序运行，A引用C，并取消对B的引用，B为垃圾。
> 3. 三色标记法继续执行，ABC最终全都为黑色，无法正确回收垃圾。

**怎么解决？**

虽然可以用STW(Stop The World)，但是过于影响性能，导致程序卡顿，所以这里我们就可以引入我们的**屏障技术**

#### 屏障技术

为了在并发中的正确标记垃圾，有两种三色不变性需要满足：

- 强三色不变性 — 黑色对象不会指向白色对象，只会指向灰色对象或者黑色对象；
- 弱三色不变性 — 黑色对象指向的白色对象必须包含一条从灰色对象经由多个白色对象的可达路径

**Dijkstra 插入写屏障**：在用户执行**写**操作的时候，即修改了指针的指向，此时会触发写屏障将被指向的对象标记为黑色，保证正确被删除，虽然简单且实现了强三色不变性，但是也有一定局限性，比如说已经被标记为灰色的对象，在取消引用之后不会被标记为垃圾。

**Yuasa 删除写屏障**：在用户执行**删**操作的时候，即删除了A对B的引用，如果此时B为白色，那么就会将其标记为灰色，因为此时这个B还有一定可能被使用，不能直接删除，所以通过这个屏障，保证了弱三色不变性，防止被某一对象被错误回收，但是局限性依旧和上面的插入写屏障一样，可能会导致垃圾无法正确被回收。

#### 垃圾收集的方法

1. **增量垃圾收集**：配合三色标记法，不使用STW的方案，而是将STW分成n个时间片，与应用程序交替执行，同时也要开启写屏障，相比STW，性能肯定是更好了。
2. **并发垃圾收集**：开启读写屏障，利用多核优势与程序并行，能够最大程度的减少对应用程序的影响，但是并发垃圾收集并不总是并发的，而且在部分阶段还是会暂停应用程序的。同时，并发执行垃圾收集引起的开销也是不能忽略的一点。

**说了这么多，Go的垃圾收集器是怎么实现的？**

### Go中的垃圾收集器(GC)

#### **混合写屏障**

结合了删除写和插入写屏障，同时，如果当前栈还没有被扫描，新分配的对象标记为黑色，删操作，标记被删对象为灰色，由此一来，我们的在栈上的新对象在分配时都会自动被标记为黑色，所以不需要`Stop The World`去扫描整个栈，同时，也只有在栈未扫描是才会做标记，避免了无用的操作。

#### **垃圾收集过程**

1. 暂停程序，确保没有运行中的goroutine干扰，如果当前GC是强制触发，还需要清理还未被清理的内存管理单元，然后将状态切换为`_GCmark`，进入标记状态，随后开启用户协助程序和写屏障，并将根对象入队。
2. 恢复程序，并发地开始标记。
3. 暂停程序，确保不会再有对象改变，状态切换至 `_GCmarktermination` 并关闭辅助标记的用户程序，并清理处理器上的线程缓存。
4. 将状态切换至 `_GCoff` 开始清理阶段，关闭写屏障。
5. 恢复程序，新创建的对象标记为白色(不会影响当前GC)，后台并发回收垃圾，当goroutine申请新的内存管理单元就会触发清理。

#### **垃圾收集的触发方式**

**前情提要**：所有出现 [`runtime.gcTrigger`](https://draveness.me/golang/tree/runtime.gcTrigger) 结构体的位置都是触发垃圾收集

- 程序启动时，会启动一个goroutine：`go forcegchelper()`，

  这个goroutine负责强制触发垃圾回收，简单来说，就是调用 [`runtime.gcStart`](https://draveness.me/golang/tree/runtime.gcStart) 尝试启动新一轮的垃圾收集，但是并不只是单纯的for循环，这个goroutine在循环中会调用 [`runtime.goparkunlock`](https://draveness.me/golang/tree/runtime.goparkunlock) 主动陷入休眠，大多数时间，这个goroutine都是睡眠的状态，如何将它唤醒呢？

  之前提到的系统监控， [`runtime.sysmon`](https://draveness.me/golang/tree/runtime.sysmon) 会构会构建 [`runtime.gcTrigger`](https://draveness.me/golang/tree/runtime.gcTrigger) 并调用 [`runtime.gcTrigger.test`](https://draveness.me/golang/tree/runtime.gcTrigger.test) 方法判断是否需要触发垃圾回收，如果需要触发，系统监控会将 [`runtime.forcegc`](https://draveness.me/golang/tree/runtime.forcegc) 状态中持有的 Goroutine 加入全局队列等待调度器的调度。

- 手动触发，用户程序会通过 [`runtime.GC`](https://draveness.me/golang/tree/runtime.GC) 函数在程序运行期间主动通知运行时执行，该方法在调用时会阻塞调用方直到当前垃圾收集循环完成，在垃圾收集期间也可能会通过 STW 暂停整个程序。

- 申请内存时，之前**内存分配器**的章节提到了微对象，小对象，大对象，这三类对象创建时都可能触发垃圾回收

#### **垃圾收集的运行过程**

之前提到了垃圾收集的大致运行过程和触发方式，接下来细说一下：

1. 在启动垃圾收集的时候，会调用 [`runtime.gcStart`](https://draveness.me/golang/tree/runtime.gcStart)方法，这很复杂，显然不能一句话解决，首先会调用 [`runtime.gcTrigger.test`](https://draveness.me/golang/tree/runtime.gcTrigger.test) 检查是否满足了垃圾收集的条件，与此同时，还会调用 [`runtime.sweepone`](https://draveness.me/golang/tree/runtime.sweepone) 清理已经被标记的内存单元，**试图**清理已经被标记的内存单元，并不一定会清理完，当然，这只是开始。
2. 当验证完成了，会调用 [`runtime.stopTheWorldWithSema`](https://draveness.me/golang/tree/runtime.stopTheWorldWithSema) 来暂停程序(这是真的StopTheWorld)，并调用 [`runtime.finishsweep_m`](https://draveness.me/golang/tree/runtime.finishsweep_m) 保证清理完成之前的被标记的内存单元的工作，**为什么会有之前的垃圾没有清理？**这是因为我们之前GC完成之后，仅仅是将垃圾标记了出来，并没有真正的去清理它，而真正的去清理它需要在一定条件下触发，并且是并发运行的，这种情况下，我们第二次GC的时候，垃圾很可能是没有清理完的，所以我们需要去将上一次遗留的垃圾清理掉。
3. 在完成了全部的准备工作，接下来就是执行了，此时我们会将全局的垃圾收集状态修改到 `_GCmark` ，然后进行一系列初始化我们的标记环境(这里初始化了什么环境可以回到原书去看看)，最后会重启我们的程序，在后台并发地去标记我们的事件了。
4. **后台标记干了些什么？**在上一步完成之后，我们的程序会调用 [`runtime.gcBgMarkStartWorkers`](https://draveness.me/golang/tree/runtime.gcBgMarkStartWorkers) 启动与M数量对等的标记任务goroutine，这些goroutine不会无意义的循环，而是会陷入休眠等待调度器的唤醒，执行标记任务的goroutine有三种模式，**专用模式**(独占处理器)，**分时模式**(cpu使用率低，启动该类型goroutine使其达到利用率)，**空闲模式**(当前处理器没有可以运行的goroutine，会运行垃圾收集的任务直到被抢占)
5. 当然，每一种模式都会去调用 [`runtime.gcDrain`](https://draveness.me/golang/tree/runtime.gcDrain) 扫描并标记所有可达对象，一旦所有的工作都陷入等待，就可以认为标记工作已经完成，此时会调用 [`runtime.gcMarkDone`](https://draveness.me/golang/tree/runtime.gcMarkDone)，除此之外，写操作都会调用 [`runtime.gcWriteBarrier`](https://draveness.me/golang/tree/runtime.gcWriteBarrier)，也就是写屏障；而新创建的对象，则是通过调用 [`runtime.gcmarknewobject`](https://draveness.me/golang/tree/runtime.gcmarknewobject) 完成标记，这部分还有更详细的讲述，可以看[原文的这个地方](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/#%e5%86%99%e5%b1%8f%e9%9a%9c)。
6. 最后，当所有可达对象都被标记后，该函数会将垃圾收集的状态切换至`_GCmarktermination`，当然如果本地队列中还存在没有处理完的任务，则会被放入全局队列等待处理。随后所有的任务都处理完成，我们便会进行我们标记阶段最后的处理，关闭写屏障，切换状态，唤醒所有协助垃圾收集的用户程序，恢复goroutine的调度并且调用 [`runtime.gcMarkTermination`](https://draveness.me/golang/tree/runtime.gcMarkTermination) 进入标记终止阶段。
7. 最后的最后，我们会在一定条件下并发执行垃圾的清理工作，我们之前的标记阶段，主要是为了将垃圾标记出来，而现在主要的任务就是将垃圾清理了，但是此时并不会主动的去调用GC进行垃圾处理，而是在一定条件下会执行相应的垃圾处理，比如说我们再执行一次GC，这就是一种触发垃圾回收的条件，还有就是用户申请内存的时候也会触发垃圾的回收，而所有的回收工作最终都是靠 [`runtime.mspan.sweep`](https://draveness.me/golang/tree/runtime.mspan.sweep) 完成的。

#### **Other**

垃圾回收工作会在扫描的时候，将扫描的对象分配给不同的P维护的gcWork，以此来提高垃圾处理的并行性，尽管如此，也可能会遇到负载不均衡的问题。这个时候，当一个P维护的gcWork的持有的任务过多时，[`runtime.gcWork.balance`](https://draveness.me/golang/tree/runtime.gcWork.balance) 会将P的部分任务放到全局队列。

**标记辅助技术**：

该技术是为了防止应用程序分配内存的速度超出标记的速度，标记辅助的原则是：

>  **分配多少内存，就需要完成多少标记任务**。

具体来讲，每一个goroutine在分配内存的时候，都相当于**欠下了债务**，如果它的债务欠的太多，变成了负数，这个goroutine就需要暂停自己的任务，去执行标记工作，直到**债务**还清(弥补了自己分配的内存大小的标记)，但是如果在GC负载很低的情况下，这不是会拖慢goroutine的运行吗？实际上，这里还有一个**全局信用**字段，如果当前GC负载低，goroutine分配内存的速度远比不上GC标记的速度，那么此时的**全局信用**的值就会很高，否则，就会很低，当一个goroutine欠下债务的时候，会首先去用**全局信用**来弥补，如果全局信用不够，那么只能这个goroutine自己去执行标记工作了。

由此一来，就实现了高吞吐，低延迟的平衡。



----

## 11. 栈内存管理

### 逃逸分析

在当前函数返回的时候，其中的本地变量会被回收，而下面这个C语言的函数例子，就是错误的返回方式，由于i在函数返回时会被回收，所以当前返回的指针是不合法的，显然，这个语法也通不过编译阶段。

```c
int *dangling_pointer() {
    int i = 2;
    return &i;
}
```

回到Go语言，go的编译器使用**逃逸分析**来确定当前的内存分配应该分配到栈上，还是堆上(包括new，make和字面量等方法隐式分配的内存)，由此一来，便能很轻松的避免上面的问题。

Go 语言的**逃逸分析**遵循以下两个不变性：

> 1. 指向栈对象的指针不能存在于堆中。
> 2. 指向栈对象的指针不能在栈对象回收后存活。

**逃逸分析**是静态分析的一种，而**静态分析**不是我们今天的重点，这里简要介绍一下，就是在不运行代码的情况下，分析程序的结构、逻辑和可能行为的方法。它通常在**编译期**完成，通过检查代码的语法、类型、数据流、控制流等信息，发现可能的错误、优化点或安全隐患。

在编译器解析了整个go语言源文件之后，会构造出一个**抽象语法树**，编译器则可以通过这个抽象语法树来分析数据流，然后构建一个有向图表示变量间的分配关系，然后根据**逃逸分析**的不变性，决定变量分配到栈还是堆。

### 栈的结构

**分段栈**：

分段栈是Go 语言在 v1.3 版本之前的实现，栈空间通过链表的形式串起来，优点是能够按需为当前goroutine分配内存，并且可以及时减少内存占用。

但是缺点也很明显，**热分裂**，如果当前栈已经接近满的状态，而在一个循环里面反复调用一个函数，此时就会不断触发栈扩容和栈释放，造成极大的消耗，并且，在触发栈扩容收缩的时候，都会出现额外的工作量。

**连续栈**：

当前Go语言的实现，初始大小为2KB，当栈空间不足时，分配一个更大的栈空间，并将原栈的数据拷贝到新分配的栈中，会经历以下几个步骤：

> 1. 分配新的栈空间。
> 2. 旧栈数据复制到新栈。
> 3. 将指向旧栈中的数据的指针指向新栈中的数据。(放在堆上的元素不需要管)
> 4. 销毁并回收旧栈的空间。

连续栈由于数据的拷贝和指针的转向增加了扩容时的开销，但也解决了**热分裂**引起的性能问题，在GC回收时，如果发现当前栈只被使用了四分之一，那就将内存减小一半，这样就不会频繁的触发扩容缩容机制了。

### 工作原理

```go
type stack struct {
	lo uintptr
	hi uintptr
}
```

hi表示栈的起始位置，lo表示结束位置。

栈的内存由mspan来追踪，最后由堆进行统一管理，尽管如此，栈内存依旧遵循着先进后出的原理，也就是作为栈来使用由此一来，我们也可以理解为什么作者说可以将go中的栈内存视为在堆上进行分配的了。如果还是不太明白，可以去思考一下[`runtime.stackinit`](https://draveness.me/golang/tree/runtime.stackinit)的源代码以及其中涉及到的结构体。

当然，栈内存分配的时候也和堆一样有内存缓存策略，当分配小内存的时候，从全局栈缓存和本地缓存获取内存；分配大内存时，会优先向全局的打栈缓存 [`runtime.stackLarge`](https://draveness.me/golang/tree/runtime.stackLarge) 中获取内存空间，否则就会自己去堆上申请一块空间。

在每一次调用函数的时候，都会检查当前的栈空间是否足够，如果不够，就会触发扩容，会保存一些栈的相关数据并调用 [`runtime.newstack`](https://draveness.me/golang/tree/runtime.newstack) 创建新的栈，在这里还需要做一些准备工作，完成之后，才会真正的拷贝，调整指针需要调用 [`runtime.adjustpointer`](https://draveness.me/golang/tree/runtime.adjustpointer)，而最后通过 [`runtime.stackfree`](https://draveness.me/golang/tree/runtime.stackfree) 释放原始栈的内存空间。

而栈缩容，事实上也是通过创建新的栈，并拷贝数据来实现的。



----

## 12. 锁

这部分是我最开始看的，当时还没写笔记，索性最后写了。

### **Mutex**

**数据结构**

```go
type Mutex struct {
	state int32
	sema  uint32
}
```

state是mutex的状态，由四个字段组成：

> `mutexLocked` — 表示互斥锁的锁定状态
>
> `mutexWoken` — 表示是否有被唤醒的等待者，防止重复唤醒
>
> `mutexStarving` — 当前的互斥锁是否进入饥饿模式
>
> `waitersCount` — 当前互斥锁上等待的 Goroutine 个数

**Lock**

锁虽然存在自旋的部分，但是长时间自旋只会消耗cpu的资源，所以go语言中对自旋做出了一些限制：

- 处于普通模式
- 运行在多cpu机器上
- 当前goroutine自旋次数小于4
- 当前机器上存在一个正在运行的并且运行队列为空的P。

自旋后，还会计算当前的锁的状态(state)，随后使用CAS(Compare And Swap)来更新状态，如果更新失败，表示交换失败，此时由另一个goroutine已经更新了这个状态，所以无法更新，随后继续下一次循环。

如果此时CAS成功了，我们并不能说他就可以拿到锁了，如果当前锁未处于饥饿模式或者锁定状态，就说明锁定成功，否则进入另一条分支：**排队**，随后调用 `runtime_SemacquireMutex` 让当前 goroutine 挂起，等待信号量将他唤醒。

被唤醒之后，会判断是否处于饥饿状态，如果是，更新状态，进入下一轮循环，值得一提的是，当unlock的时候，会直接判断当时是否由处于饥饿状态的goroutine，然后才会发出信号唤醒相应的goroutine，所以这一轮设置的饥饿模式，在下一轮释放锁的时候才会生效。

**UnLock**

这部分实现逻辑相对比较简单，首先会判断当前处于饥饿模式，如果有，则会才去手递手的形式唤醒饥饿模式的goroutine，否则，直接发出信号，唤醒一个goroutine

### **读写锁**

读写锁是一种更细粒度的锁，当使用读锁时，写锁会被阻塞，而读锁不会，当使用写锁时，读写锁都会被阻塞。

这部分逻辑和**Mutex**差不多，只是多了一个读锁数量的字段，比较简单，适用于读多写少的环境，在这里不多赘述，有兴趣可以去原书阅读。

---

### **WaitGroup**

[`sync.WaitGroup`](https://draveness.me/golang/tree/sync.WaitGroup) 对外暴露了三个方法：[`sync.WaitGroup.Add`](https://draveness.me/golang/tree/sync.WaitGroup.Add)、[`sync.WaitGroup.Wait`](https://draveness.me/golang/tree/sync.WaitGroup.Wait) 和 [`sync.WaitGroup.Done`](https://draveness.me/golang/tree/sync.WaitGroup.Done)。

相信使用过wg的朋友都知道这些方法有什么用，Done仅仅是向Add方法传入了-1，这里不多说，而[`sync.WaitGroup.Add`](https://draveness.me/golang/tree/sync.WaitGroup.Add)和[`sync.WaitGroup.Wait`](https://draveness.me/golang/tree/sync.WaitGroup.Wait) 的实现也比较简单

Add方法会更新wg中的计数器的数量，与此同时，在Add方法的最后，会有一个for循环，向所有正在等待的goroutine发送一个信号来唤醒他们，举个例子，比如当传入的参数为负数，也就是Done方法，此时wg的计数为1，所有调用wait方法的goroutine陷入睡眠，然后通过传入负数，可以唤醒这些goroutine，继续工作。

Wait方法主要就是等待wg的计数器归零，陷入休眠，等待信号，一旦被唤醒，就会退出。

---

### **Once**

[`sync.Once.Do`](https://draveness.me/golang/tree/sync.Once.Do) 是 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 结构体对外唯一暴露的方法，传入一个函数，如果当前结构体的do方法已经被执行，那么则不会执行，如果还没有执行过，就会执行，简单来说，就是这个Do方法只会生效一次，**这里有一个误区**，部分人以为这里只是让某一个函数只能执行一次，实际上无论你传入什么函数，最终只会有一个函数被调用。

另外，在do方法中，还会调用doslow这个方法，在内部会加上互斥锁保证同一时刻只有一个goroutine执行。

---

### **Cond**

[`sync.Cond`](https://draveness.me/golang/tree/sync.Cond) 对外暴露的 [`sync.Cond.Wait`](https://draveness.me/golang/tree/sync.Cond.Wait) 方法会将当前 Goroutine 陷入休眠状态，这个方法内部很简单，先将计数器加一，随后被加入等待的goroutine链表，陷入睡眠，等待唤醒。

**如何唤醒？**[`sync.Cond.Signal`](https://draveness.me/golang/tree/sync.Cond.Signal) 和 [`sync.Cond.Broadcast`](https://draveness.me/golang/tree/sync.Cond.Broadcast) 就是用来唤醒陷入休眠的 Goroutine 的方法，Signal会唤醒队列最前面的一个goroutine，而broadcast，如其名，会按次序唤醒所有等待的goroutine。

感觉这玩意不常用。

**剩下的同步原语感觉真不熟**

---

### **ErrGroup**

结合了wg，once的精华，通过调用 [`golang/sync/errgroup.Group.Go`](https://draveness.me/golang/tree/golang/sync/errgroup.Group.Go) 方法启动goroutine，然后使用[`golang/sync/errgroup.Group.Wait`](https://draveness.me/golang/tree/golang/sync/errgroup.Group.Wait) 等待所有goroutine执行完毕，并查看是否存在错误。

---

### **Semaphore**

信号量，也可以用来作为流量控制之类的事情，感觉跟Sentinel挺像的。

- [`golang/sync/semaphore.NewWeighted`](https://draveness.me/golang/tree/golang/sync/semaphore.NewWeighted) 用于创建新的信号量。
- [`golang/sync/semaphore.Weighted.Acquire`](https://draveness.me/golang/tree/golang/sync/semaphore.Weighted.Acquire) 阻塞地获取指定权重的资源，如果当前没有空闲资源，会陷入休眠等待。
- [`golang/sync/semaphore.Weighted.TryAcquire`](https://draveness.me/golang/tree/golang/sync/semaphore.Weighted.TryAcquire) 非阻塞地获取指定权重的资源，如果当前没有空闲资源，会直接返回 `false`。
- [`golang/sync/semaphore.Weighted.Release`](https://draveness.me/golang/tree/golang/sync/semaphore.Weighted.Release) 用于释放指定权重的资源，同时会按照**先进先出**的策略唤醒因为无法获取足够的资源而阻塞的goroutine。

---

### **SingleFlight**

虽然作者说用这个可以处理缓存击穿的问题，~~但是感觉不太常用~~。

主要就是通过调用 [`golang/sync/singleflight.Group.Do`](https://draveness.me/golang/tree/golang/sync/singleflight.Group.Do)方法，来限制同一时间重复的请求次数，阻塞的等待参数返回，而[`golang/sync/singleflight.Group.DoChan`](https://draveness.me/golang/tree/golang/sync/singleflight.Group.DoChan)会返回一个管道，让你只会去取拿到的返回值，相当于异步执行。

常看常新，很牛逼，通过创建哈希表，删除哈希表很短时间内的延迟来实现去重，太牛逼了。

---

## 13. 哈希表

### **实现方法**

为了解决哈希冲突，我们常见的方法有：**开放寻址法**和**拉链法**。

**开放寻址法**：写数据时，计算哈希值，找到相应的槽位，如果当前数组位有其他数据，比对key值，如果一样，写入，如果不一样，依次遍历直到遇见槽位为空，或者key值相同的情况，写入，当读数据时，也是一样，计算哈希值，找到槽位并比对key值，如果一样就读取，不一样就遍历下一位；**缺点**是随着装载因子(元素数量和数组大小比值)增加，哈希表的读写性能会急剧下降，因为此时冲突基本到处都有，读写效率很低。

**拉链法**：相比于开放寻址法的全部装在一起，拉链法则是将产生哈希冲突的元素统一放在桶中，一个哈希表有很多个桶，当然，所谓的**桶**，一般是用的链表进行实现，虽然和开放寻址法一样，随着装载因子增加，性能也会下降，但是拉链法的每一个kv是链式存储的，比起而开放寻址法采取数组，是连续存储的，拉链法更复杂，但是性能更优

----

关于go中哈希表的数据结构还是在放在例子中比较好

### **读写操作**

#### **读操作**

由于v := hash[k]和v,ok := hash[k]两种读取方式在源码中的体现区别不大，就是加了一个bool值。

接受两个参数时，会使用 [`runtime.mapaccess2`](https://draven.co/golang/tree/runtime.mapaccess2)，这个函数最开始，会使用哈希表初始化时确定的哈希函数和种子获取key的哈希值，过 [`runtime.bucketMask`](https://draven.co/golang/tree/runtime.bucketMask) 和 [`runtime.add`](https://draven.co/golang/tree/runtime.add) 拿到这个k应该在的桶的循环和哈希高8位数字(用于在桶中的加速匹配，直接筛掉部分数据)，找到桶之后，遍历比对高八位，如果不同，直接跳过，如果比对成功，直接比对传入的key和该位置上存储的key是否相同，如果相同，直接返回对应的value，不同则继续比对，如果遇到了空槽位，那么就会返回nil，此处还有一个概念叫做**溢出桶**，由于每个桶默认只能装八个元素，所以多余的元素需要新建桶来实现，但是这个桶中的元素又依赖于我们刚刚找到的桶，所以他就叫做溢出桶，通过链式连接到我们的正常桶，如果在正常桶中没有找到对应的key，会跳到溢出桶进行比对，直到遇见空槽。

#### **写操作**

读取操作时，会将其转换为[`runtime.mapassign`](https://draven.co/golang/tree/runtime.mapassign) 函数的调用，与写操作时调用的函数类似。

第一步是一样的，根据key拿到对应的桶和哈希值，同读操作一样去遍历，当然，如果桶已经被装满了，就会调用 [`runtime.hmap.newoverflow`](https://draven.co/golang/tree/runtime.hmap.newoverflow) 创建新桶，新创建的桶不仅会被追加到已有桶的末尾，还会增加哈希表的 `noverflow` 计数器(记录溢出桶的数量)，另外这也表明了key在桶中是不存在的，如果存在肯定好说，直接更新value即可，不存在就会通过 [`runtime.typedmemmove`](https://draven.co/golang/tree/runtime.typedmemmove) 将key的值复制到对应的地址空间。

#### **扩容操作**

除此之外，[`runtime.mapassign`](https://draven.co/golang/tree/runtime.mapassign) 还涉及扩容的操作，随着哈希表中元素的逐渐增加，哈希的性能会逐渐恶化，所以我们需要更多的桶和更大的内存保证哈希的读写性能。

[`runtime.mapassign`](https://draven.co/golang/tree/runtime.mapassign) 函数会在以下两种情况发生时触发哈希的扩容：

1. 装载因子已经超过 6.5；
2. 哈希使用了太多溢出桶；

同时，哈希的扩容并不是原子的，所以还会判断当前哈希是否处于扩容的状态，防止二次扩容。

我们会调用[`runtime.hashGrow`](https://draven.co/golang/tree/runtime.hashGrow)来进行扩容，过程中，会通过 [`runtime.makeBucketArray`](https://draven.co/golang/tree/runtime.makeBucketArray) 创建一组新桶和预创建的溢出桶，当然，既然新创建了桶，相比于原来的哈希表查询的结果肯定也会有所不同，所以还需要对桶中的元素进行重新分配，哈希表的数据迁移的过程在是 [`runtime.evacuate`](https://draven.co/golang/tree/runtime.evacuate) 中完成的，它会对传入桶中的元素进行再分配。

扩容有两种，**等量扩容**和**增量扩容**，两种虽然不一样，但是都是为了提高查询效率，减少溢出桶的数量，将更多的元素放进正常桶中，为什么会有等量扩容？因为在哈希表删除操作kv键值对的时候，不会删除溢出桶，这就导致了无效查询，降低了效率。而增量扩容则是由于桶的数量变化，会重新计算哈希值去映射到不同的桶上。

在扩容期间发生读写操作怎么办？此时我们的hash结构体中存在 `oldbuckets` 字段，在还没有分流完的时候存储旧桶的数据，扩容期间发生读写，会优先从这里读写数据。

#### **删除操作**

go中使用delete关键字删除哈希表中的一个key，delete会被替换为 [`runtime.mapdelete`](https://draven.co/golang/tree/runtime.mapdelete) 函数簇中的一员，以成 [`runtime.mapdelete`](https://draven.co/golang/tree/runtime.mapdelete)为例，和读写操作一样，也需要先找到相对应的桶，然后找到对应的key，当然，如果此时正在扩容，先等待扩容完毕，再去执行删除，删除则是将当前位置的元素标记为emptyone，不会影响后续查找，同时并不是立即删除，而是打上标记，等待后续被垃圾收集器处理，且不会影响当前位置为emptyone的标记，不会影响后续查找。



----

## 14. 字符串

字符串string是一个只读数据，意味着我们不能修改它，但是我们可以将其转换为[]byte进行修改，再转换回去。

这种不变性意味着我们不会引用到意外发生改变的值，而go中string是不可变类型，作为哈希表的键是安全可靠的，而这也正是引用类型不能作为哈希表键的原因。

golang中，由于字符串string是只读的，所以任何拼接操作都是基于拷贝的，会在内存中新开辟一个空间存储新的字符串，但是和java一样，go中也有`strings.Builder`这个高效的字符串拼接工具，使得大字符串的拼接更加高效。

go中字符串的声明有两种，一种是双引号""单行的字符串，另一种是反引号``，支持多行，内部可以使用双引号，通常用于编写json，其他复杂数据格式会用到。

## 15. 切片

切片在数组的基础上进行了包装，提供了对数组中部分连续片段的引用，这也意味着，我们可以修改切片中的数据以及修改长度

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

len代表切片的长度，Cap则代表容量，当len超过一定范围，切片会自动扩容，与此同时，虽然切片是值传递，但是底层依旧共享一个数组，另外在使用[:]的时候，也不是深拷贝，而是获取了一份指向了底层数组的引用，同样的，在函数传递的时候，也是一样，是得到了一份原切片的拷贝，但是指向的底层数组依旧是一样的，区别就是，这个拷贝的容量和长度不会影响到原切片，当然，如果传递的是切片指针，结果很显然是不一样的。

值得注意的是，虽然append会返回一个新的切片，与原来的切片无关，但是如果使用二维切片，比如

```go
ans = append(ans, tmp)
```

此时如果得到的得到的二维切片，实际上内部追加的是对tmp的引用，此时如果对tmp进行修改，那么ans中的的数据也会受到影响，正确的做法应该是使用`copy(a, b)`或者

```go
ans = append(ans, append([]int{}, tmp...))
```

通常，copy性能更加高效，这样，就能得到一个全新的切片。(写算法题的时候一定要注意😡)

另外刚刚我们提到了切片的拷贝，当我们指向`copy`时，编译器转换的函数中的 [`runtime.memmove`](https://draven.co/golang/tree/runtime.memmove) 会负责拷贝内存到目标区域。

另一个值得一提的就是扩容的问题，虽然有很多公式，但是我发现在部分情况下，扩容的公式似乎不太管用，总之，在最开始切片容量比较小的时候，扩展的容量大多是两倍扩容，到了之后，扩容的容量就会慢慢降低比例，防止空间的浪费。



---

## 16. rune和byte的区别？

`byte` 是 `uint8` 的别名，本质上是一个 8 位无符号整数（0~255），通常用于表示单个字节，用于操作ACSII字符或二进制数据(比如文件，网络流)

`rune` 是 `int32` 的别名，本质上是一个 32 位有符号整数。可以存储所有的 Unicode 字符，适用于处理 **多字节字符**（如汉字、Emoji）



## 17. Go中的Swap常用什么操作来实现？

我们在代码中经常能够看见：

```go
	nums[i], nums[j] = nums[j], nums[i]
```

这样的代码，这就是我们的swap，在其他部分语言中，没有这样并行赋值的语法，而在Golang中，是支持的，而我们熟悉汇编的朋友可能会疑惑，这个转换成汇编代码是什么样子的？会不会先更新`nums[i]`，然后利用`nums[i]`去更新`nums[j]`？这里实际上是先将等号右边的数值先计算出来，保存下来，然后将他们的值赋给左边的变量，所以是不会有上述问题的，这样，我们的swap实现实际上是更加简洁优雅的，并且这样的语法能够轻松的应付很多场景。

## 18. defer和return执行顺序？

return会先给返回值赋值，然后执行defer执行收尾操作，最后才执行函数返回，如何测试？比方说：

```go
func test() int {
    result := 1
    defer func() {
        result++
        fmt.Println("defer:", result)
    }()
    return result
}
```

他的最终打印的result为2，返回的result为1.

## 19. Goroutine是用户态的切换，但是如何切换指令指针的？

很简单，该书中，已经提过了这一段代码，我们是通过 jmp 指令间接的去修改我们的指令指针的，所以可以确确实实地在用户态就可以实现切换操作，而不需要陷入内核，尽管操作系统中的内核中也是 jmp 实现的跳转，这个没啥好说的，但是学过os的理解确实不一样了一点，就简单提一下。

```assembly
	MOVL gobuf_pc(BX), BX  // 获取待执行函数的程序计数器
	JMP  BX                // 开始执行
```

## 20. 凭啥 sync.Map 这么快？

偶然看见一份面经是问为什么 sync.Map 更快，这引起了我的好奇，我以前几乎没有使用过这个并发安全的哈希表，仅仅是单纯的给原生哈希表加锁，于是，我想去看看 sync.Map 的源码。

我测试出来是在高并发读的情况下， sync.Map 的速度远大于读写锁的 Map ，测试如下：

```go
func Test_syncmap(t *testing.T) {
	mp := sync.Map{}
	wg := sync.WaitGroup{}
	n := 10000
	for i := 0; i < n; i++ {
		wg.Add(1)
		go func() {
			for i := 0; i < n; i++ {
				_, _ = mp.Load(i)
			}
			wg.Done()
		}()
	}
	wg.Wait()
}

func Test_lockmap(t *testing.T) {
	mp := map[int]int{}
	lock := sync.RWMutex{}
	wg := sync.WaitGroup{}
	n := 10000
	for i := 0; i < n; i++ {
		wg.Add(1)
		go func() {
			for i := 0; i < n; i++ {
				lock.RLock()
				_, _ = mp[i]
				lock.RUnlock()
			}
			wg.Done()
		}()
	}
	wg.Wait()
}
```

我们的测试结果如下：

```bash
$ go test -bench . -benchmem -v 

=== RUN   Test_syncmap
--- PASS: Test_syncmap (0.07s)
=== RUN   Test_lockmap
--- PASS: Test_lockmap (2.07s)
PASS
ok      github.com/Rinai-R/Some-WORKs/2025/05May/20250505       2.148s
```

明显看到， sync.Map 快了几个数量级，但是在高并发读的情况下，两者实际上的速度是差不多的，甚至 sync.Map 还要更差，所以应该在适当的时候选择适当的 map ，下面从源码角度看看为什么 sync.Map 这么快吧。

先来看看它的数据结构：

```go
// Map 是一个基于 HashTrieMap 实现的线程安全的映射（map）。
// 它封装了一个泛型的 HashTrieMap，并使用原子操作和锁机制来确保并发访问的安全。
type Map struct {
	// _ 是一个无用的字段，用于阻止该结构体被复制（通过结构体的 "noCopy" 类型实现）。
	// 它是 Go 中防止结构体被意外复制的常用技术。
	_ noCopy

	// m 是实际存储键值对的 HashTrieMap。
	// HashTrieMap 是一个基于哈希前缀树的数据结构，用于提供支持并发操作的键值存储。
	m isync.HashTrieMap[any, any]
}
```

我们可以进一步看看 HashTrieMap 的数据结构：

```go
// HashTrieMap 是一个哈希前缀树（Hash Trie）实现，支持高效的键值存储。
// 它是线程安全的，并且通过使用原子操作和分段锁来提高并发性能。
type HashTrieMap[K comparable, V any] struct {
	// inited 表示 HashTrieMap 是否已经初始化。
	// 使用原子操作来保证并发环境下的安全性。
	inited atomic.Uint32

	// initMu 是用于初始化时的互斥锁，防止并发初始化。
	// 只有一个线程可以完成初始化过程。
	initMu Mutex

	// root 存储 HashTrieMap 的根节点指针。
	// 使用 atomic.Pointer 来保证并发访问时的原子性和一致性。
	root atomic.Pointer[indirect[K, V]]

	// 哈希函数，用于计算哈希值
	keyHash hashFunc

	// valEqual 是用于比较值是否相等的函数。
	valEqual equalFunc

	// seed 是用于生成哈希值的种子，提供一定程度的随机性。
	// 可以在哈希冲突情况下优化性能。
	seed uintptr
}
```

很多字段我们并不能仅凭名字就可以推断出它的作用，但是部分字段还是和原始的哈希表是一致的，比如 seed/KeyHash ，但都不是我们今天的重点，想要知道这些字段有什么作用，我们可以进一步去看看 sync.Map 的方法，比如 Store ， Store 的作用是将键值对存储在这个哈希表中，除去一些封装，他在最后会调用一个 swap 的方法，这也是最核心的部分，我们也可以了解到为什么 sync.Map 在写多读少的场景下很慢：

```go
// 该函数交换给定键的值，并返回旧值（如果有的话）。
// 通过返回的 loaded 标志来报告键是否存在于映射中。
// 在实现过程中，我们首先查找键的位置，如果找到，则交换旧值并返回。
// 如果未找到键，则进行插入操作。
func (ht *HashTrieMap[K, V]) Swap(key K, new V) (previous V, loaded bool) {
	// 初始化 HashTrieMap（如果尚未初始化）
	ht.init()

	// 计算给定 key 的哈希值
	hash := ht.keyHash(abi.NoEscape(unsafe.Pointer(&key)), ht.seed)

	var i *indirect[K, V]        // 当前节点
	var hashShift uint           // 用于控制哈希位移的变量
	var slot *atomic.Pointer[node[K, V]] // 当前插槽位置，其中存储的是 entry/node 的指针
	var n *node[K, V]            // 当前节点

	// 迭代查找键的位置或插入的候选位置
	for {
		// 从根节点开始加载
		i = ht.root.Load()
		hashShift = 8 * goarch.PtrSize
		haveInsertPoint := false

		// 遍历哈希位进行查找
		for hashShift != 0 {
          // 这里是 - 4
			hashShift -= nChildrenLog2

			// 通过掩码计算出当前的对应的儿子是谁
          // hashShift 每个循环都会更新
          // nChildrenMask 为 15 ，只会保留 0 ~ 15 的数字
          // 每个父节点只有 16 个儿子，刚刚好
			slot = &i.children[(hash>>hashShift)&nChildrenMask]
			n = slot.Load()

			// 如果当前插槽为空或是一个现有条目，我们找到插入点或替换位置
			if n == nil || n.isEntry {
				haveInsertPoint = true
				break
			}
			// 如果当前节点不是目标节点，继续查找
			i = n.indirect()
		}

		// 如果没有找到合适的位置，则报错
		if !haveInsertPoint {
			panic("internal/sync.HashTrieMap: ran out of hash bits while iterating")
		}

		// 加锁，双重检查我们看到的内容
		i.mu.Lock()
		n = slot.Load()
		// 如果插槽为空或者是一个条目，并且节点未死亡，则可以继续插入
		if (n == nil || n.isEntry) && !i.dead.Load() {
			break
		}
		// 否则需要重新开始查找
		i.mu.Unlock()
	}

	// 解锁
	defer i.mu.Unlock()

	var zero V
	var oldEntry *entry[K, V]
	if n != nil {
		// 如果找到了现有节点且键匹配，则进行值交换
       // 当然也可能是哈希碰撞了，这里也会处理这个情况
		oldEntry = n.entry()
		newEntry, old, swapped := oldEntry.swap(key, new)
		if swapped {
			// 如果交换成功，更新插槽并返回旧值
			slot.Store(&newEntry.node)
			return old, true
		}
	}

	// 如果键不匹配，则执行插入操作
	newEntry := newEntryNode(key, new)
	if oldEntry == nil {
		// 原子地存储数据
		slot.Store(&newEntry.node)
	} else {
		// 这里说明哈希碰撞了，并且没有在溢出链中找到相同的 key 节点，只能新加一个链
		slot.Store(ht.expand(oldEntry, newEntry, hash, hashShift, i))
	}

	// 如果没有找到现有条目，返回零值和 loaded 为 false
	return zero, false
}
```

可以看到，我们的 sync.Map 在 Linux 的实现中是以哈希前缀树组织的，其中每个节点的数据结构如下：

```go
// 哈希前缀树中的一个内部节点。哈希前缀树使用该节点来组织哈希表中的数据。
// 它包含子节点、父节点，并且用于管理树的层级结构。
// 该节点用于存储指向子节点的指针，并通过锁机制来保护对数据的修改。
// 具体的结构包括节点数据、锁和原子标记，保证在并发环境下的安全访问。

type indirect[K comparable, V any] struct {
    node[K, V]                     // 包含节点的一些信息
    dead     atomic.Bool           // 标记该节点是否被删除，用于垃圾回收或标记死节点
    mu       Mutex                 // 用于保护对 children 的修改，以及包含条目的节点
    parent   *indirect[K, V]       // 指向该节点的父节点，帮助管理树的层级结构
    children [nChildren]atomic.Pointer[node[K, V]] // 当前节点的子节点指针数组，共有 16 个子节点
}
```

我们发现，我们的这个 indirect 并没有保存键值对，所以除了这个之外，我们还使用了一个 entry 结构体来存储这个节点的具体数据，也就是第一个成员数据：

```go
// entry 是哈希前缀树（hash-trie）中的一个叶子节点（leaf node）。
// 它存储了一个键值对，并且还支持处理哈希碰撞。
type entry[K comparable, V any] struct {
    node[K, V]                     // 嵌套的 node 结构体，包含节点的一般信息。
    overflow atomic.Pointer[entry[K, V]] // 处理哈希碰撞的溢出指针。使用原子指针确保线程安全。
    key      K                     // 存储的键，类型为 K，必须是可比较的（comparable）。
    value    V                     // 存储的值，类型为 V，可以是任意类型（any）。
}
```

其中，存在一个 node 结构体，他实际上是存储了这个节点的类型，表示他是否为一个 entry ，事实上，我们的 node 结构体有一个方法就是可以直接得到这个 node 对应的 entry ，十分方便，因为他们在地址上指向一个位置。

我们可以知道最新版的 go 中， sync.Map 换成了哈希前缀树的实现方式来提高性能，我们的 node 的指针既可以得到 entry ，也可以得到 indirect，这里没有冲突的原因，应该是一个 node 只能转换成一种结构体，如果 node 的 isentry 为 true ，则可以转换为 entry ，否则只能转换为 indirect 。（个人理解）

除此之外，由于每个节点都有 16 个儿子，所以根据哈希值去检索会非常快，在处理哈希碰撞的时候，则是采取 entry 里面的 overflow 字段，我们对应的函数处理如下：

```go
// swap 替换溢出链中的条目，如果键相等则交换值。
// 返回新的条目链、旧值，以及是否进行了交换。
// swap 必须在 indirect 节点的互斥锁下调用，
// 因为 `head` 是 indirect 节点的子节点。
func (head *entry[K, V]) swap(key K, new V) (*entry[K, V], V, bool) {
    // 如果当前条目的键和目标键匹配，执行值替换
    if head.key == key {
        // 创建一个新的条目节点，并替换为新的值
        e := newEntryNode(key, new)

        // 如果头节点有溢出链，更新新的条目的溢出指针
        if chain := head.overflow.Load(); chain != nil {
            e.overflow.Store(chain) // 保留现有的溢出链
        }

        // 返回新的条目链，新值以及标记为交换成功（true）
        return e, head.value, true
    }

    // 如果当前条目的键不匹配，则遍历溢出链查找目标键
    i := &head.overflow // 获取溢出链的指针
    e := i.Load()       // 加载第一个溢出链条目

    // 遍历溢出链中的条目
    for e != nil {
        if e.key == key {
            // 找到匹配的条目，创建一个新的条目并替换值
            eNew := newEntryNode(key, new)
            eNew.overflow.Store(e.overflow.Load()) // 更新新的溢出链指针
            i.Store(eNew) // 更新当前溢出链为新的条目

            // 返回当前头节点，旧值以及交换成功标志（true）
            return head, e.value, true
        }

        // 如果未找到匹配键，继续检查下一个溢出链
        i = &e.overflow
        e = e.overflow.Load()
    }

    // 如果遍历完溢出链没有找到匹配的键，返回零值和加载标志（false）
    var zero V
    return head, zero, false
}
```

如上，如果 key 相同，则进行值交换，如果不同则说明是哈希碰撞，在溢出链中更新指针，如果溢出链中仍然找不到，返回加载标志，将在返回的函数中进行加载。

其实最开始想搞懂的是 1.24 版本之前的有 read-dirty 模式的 sync.Map 结果却发现我的版本太新了，索性就看看新的实现了，其实性能还是可以的，而原来的版本的写入性能较差应该是因为将所有的数据都放在了 dirty-map 中，而 read-map 是只读的数据，在使用 dirty-map 的时候需要加锁，任何操作都会优先从 read-map 中读取数据，我们可以将他理解为缓存，他是无锁的，所以是非常快的，而我们的 dirty-map 都会加上锁，因为他直接使用了原始的哈希表，所以必须加上锁，这个 dirty-map 可以理解为数据库，所以，对于写入操作很多的情况下，新增键值对会导致大量的加锁操作，并且在此之前还需要各种检查，同时我们读取缓冲区的时候，也会有大量的缓存失效，自然而然速度就不如 lock + map 了。（这部分是看的[这篇知乎文章](https://zhuanlan.zhihu.com/p/22878835460)，写得很详细👍）

而相比之下，我们的新版的 sync.Map 舍去了这个缓存的策略，并且舍去了哈希表这一不安全的数据结构，完全自行设计了一个并发安全的数据结构，使得读写速度更快，对于哈希前缀树实现的 sync.Map ，我们的读操作，实际上也是无锁的：

```go
func (ht *HashTrieMap[K, V]) Load(key K) (value V, ok bool) {
    // 初始化哈希表（如果没有初始化）
	ht.init()

	// 计算键的哈希值
	hash := ht.keyHash(abi.NoEscape(unsafe.Pointer(&key)), ht.seed)

	// 从根节点开始查找
	i := ht.root.Load()

	// 设置初始的 hashShift
	hashShift := 8 * goarch.PtrSize

	// 逐步遍历哈希表中的每一层，直到找到目标节点或哈希位数耗尽
	for hashShift != 0 {
		// 每次循环减小 hashShift，直到找到目标层
		hashShift -= nChildrenLog2

		// 根据当前 hashShift 计算当前节点的子节点
		n := i.children[(hash>>hashShift)&nChildrenMask].Load()

		// 如果当前节点为空，表示没有找到值，返回空值和 false
		if n == nil {
			return *new(V), false
		}

		// 如果当前节点是一个有效的键值对，返回该键对应的值
		if n.isEntry {
			return n.entry().lookup(key)
		}

		// 如果当前节点不是键值对，继续向下遍历
		i = n.indirect()
	}

	// 如果哈希位数耗尽，抛出 panic 异常，表示结构异常
	panic("internal/sync.HashTrieMap: ran out of hash bits while iterating")
}
```

简单总结一下，最新版的哈希表通过哈希前缀树实现了并发安全的哈希表，并仅仅在使用写操作的时候会涉及到一些细粒度的锁，且很多地方都是使用的原子操作来读取，同时在读操作的时候完全是无锁的，最后，附上高并发写的时候，两个版本的哈希表的 benchmark 对比：

```bash
$ go test -bench .
PASS
ok      github.com/Rinai-R/Some-WORKs/2025/05May/20250505       63.524s
$ go test -bench .
PASS
ok      github.com/Rinai-R/Some-WORKs/2025/05May/20250505       24.114s
```

测试代码：

```go
func Test_syncmap(t *testing.T) {
	wg1 := sync.WaitGroup{}
	mp1 := sync.Map{}
	n := 20000
	for i := 0; i < n; i++ {
		wg1.Add(1)
		go func() {
			for i := 0; i < n; i++ {
				mp1.Store(i, i)
			}
			wg1.Done()
		}()
	}
	wg1.Wait()
}
```

