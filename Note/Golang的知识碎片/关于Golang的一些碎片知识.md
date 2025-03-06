# 关于Golang的一些碎片知识

因为平时看的时间比较杂，没有系统的学Golang，最近在看Go语言设计与实现，感觉一些有趣，有难度的知识点就在这里用自己的话总结一下，写下来了

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

>1. 因为目前的计时器由网络轮询器管理和触发，它能够让网络轮询器立刻返回并让运行时检查是否有需要触发的计时器。						   --《网络轮询器》部分
>2. 这里将分析器的触发过程，Go 语言会在两个模块触发计时器，运行计时器中保存的函数：
>   - 调度器调度时会检查处理器中的计时器是否准备就绪；
>   - 系统监控会检查是否有未执行的到期计时器；		--《定时器》部分

最开始让我感觉很矛盾，事实上，我认为这些触发是相辅相成的，调度器作为主导的触发计时器的组件，而系统监控作为辅助，网络轮询器作为辅助，当网络轮询器陷入`poll_wait`状态时，当前M线程陷入休眠，此时这个M陷入系统调用，不会去执行其他的goroutine，与其让这个M睡死，不如直接唤醒它，让他来执行timer，也可以避免另外两种触发timer的方法会创建新的M造成的资源浪费。

---



## 4. 网络轮询器

**阻塞 I/O 模型**：

发出一个HTTP请求或者读写文件时，程序就会陷入阻塞状态，直到这个操作结束。**举个例子**，当对文件执行read系统调用时，应用程序会从用户态陷入内核态，内核会检查**文件描述符**是否可读；当**文件描述符**中存在数据时，操作系统内核会将准备好的数据拷贝给应用程序并交回控制权。

**非阻塞I/O**：

第一次发出http或者读写文件请求时，会向文件描述符读取数据，并且立即返回`EAGAIN` 错误，这表示文件描述符还在等待缓冲区的数据，也就是没准备好的意思，在第一次发出请求之后，应用程序会不断轮询调用这个请求，直到返回正确的值，此时就可以读取数据进行操作了。**为什么要这样做？**因为像第一种I/O模型，有阻塞状态，浪费了大量时间去等待，而非阻塞I/O能够立刻返回，在等待的过程中，去执行其他的操作，提高CPU利用率。

**文件描述符**：

一个I/O操作，一个tcp连接一个websocket连接或者一个http请求的抽象的表示，用来区分这些请求，方便在唤醒这些事件时执行正确的操作。(**tips**：HTTP请求可能会存在复用文件描述符的情况，但是与之对应的还有`Stream ID`来区分它们，具体的步骤是这样的：**1.**网络轮询器检测到该文件描述符可读。**2.**用 `netpoll`读取文件描述符数据。**3.**发现该描述符属于http请求，使用http解析器解析`Sream ID`来区分不同的请求，并分发给goroutine)

**GO中的网络轮询器**：

是一个全局的对象，将所有的文件描述符用一个链表来维护，可以实现快速的插入操作，至于查找操作，在发送I/O请求或者建立HTTP连接的时候(注册fd文件描述符)的时候，还会向epoll注册一份事件，此时也会保存对应fd节点的指针，当监听到返回的时候，则会返回这个指针，以此来实现快速查找。

**tips**：

在网络轮询器初始化时，会涉及到一个pipe管道的初始化，还会创建一个epoll的文件描述符fd用于监听所有的事件，而在之后，在一定条件下会调用[`runtime.netpollBreak`](https://draveness.me/golang/tree/runtime.netpollBreak)方法，这个方法会通过这个管道发送中断等待某一事件的信号，然后让这个`epoll`中进入到`poll_wait`阻塞的M去处理其他的事件。

之前提到了向epoll注册事件，实际上他是如何实现的轮询操作的呢？事实上，在轮询的过程中，如果发现某一个事件没有准备好，则会调用gopark让出这个goroutine，陷入休眠状态，此后不会对其进行无用的轮询，从而减少CPU的损耗，而epoll负责监控事件，只有在事件发生时，才会通知Go程序然后来唤醒这个goroutine，同时，调用netpoll方法的时候，如果当前没有待处理的事件，就会陷入阻塞，也就是`epoll_wait`，这可能会阻塞很久，如果休眠的时候有timer到期了，但是此时其他的所有的线程都陷入了休眠，怎么办？

这个时候就会通过我们之前初始化的管道，调用`runtime.netpollBreak`方法发出中断信号，唤醒这个M去执行这个timer，另外，当有goroutine需要执行，但是找不到M的时候，此时也会调用`runtime.netpollBreak`发出信号，让`epoll_wait`返回，来执行这个goroutine。

**重新理一下发出I/O请求的思路**，当在一个goroutine中发出I/O请求的时候，将fd注册到epoll中，同时当前goroutine被阻塞，等待fd可读，此时，当前goroutine陷入休眠，线程M执行其他goroutine，直到epoll监听到这个fd可读，再将这个goroutine放入运行队列，等待被执行。

**epoll是谁在执行？**除了之后我们会提到的系统监控，当线程M没有待运行的goroutine的时候，我们的M会调用netpoll来监听fd，此时还会传入一个超时时间，这个时间由最早过期的timer来决定，保证不会错过定时任务，但是还有一种情况，就是如果当前没有timer，也没有goroutine在M上运行，于是M启动了netpoll，陷入阻塞，那么之后资源紧张起来，新创建的goroutine找不到可供使用的M怎么办？此时`runtime.netpollBreak`就被会调用，向管道中发出信号，来终止`poll_wait`的阻塞，以此来让goroutine得到M资源。

**截止日期**：为每个fd设置截止日期，每个fd都有一个对应的timer，超时则执行回调函数取消该事件，并唤醒goroutine，使其做出相应的处理(重试或取消)

其实这部分光看《Go语言设计与实现》确实能够理解不少，毕竟是带着你去读源码，但是看了一遍这部分，再自己去读一下源码，也会别有一番收获。



----

## 5. Channel

Go中的 Channel 收发操作均遵循了先进先出的设计：

- 先从 Channel 读取数据的 Goroutine 会先接收到数据。
- 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利。

虽然目前go希望能够实现无锁的channel来提高性能，但是目前底层的channel依旧使用锁(Lock)来维护channel队列先进先出的特性。

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

**Other：标记辅助**

明天再写~

----

