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

表面上是切片，内部使用四叉堆维护timer(计时器)，堆顶元素是距离截止时间最近的计时器，四叉堆，顾名思义，就是每个节点只能有四个子节点，Go语言通过网络轮询器监听最近到期的timer是否到期，如果到期，则会执行回调。网络轮询器实际上还负责I/O事件和网络连接，如果这三者同时发生，timer执行的优先度是最低的。

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



----

## 4. 网络轮询器

**阻塞 I/O 模型**：发出一个HTTP请求或者读写文件时，程序就会陷入阻塞状态，直到这个操作结束。**举个例子**，当对文件执行read系统调用时，应用程序会从用户态陷入内核态，内核会检查**文件描述符**是否可读；当**文件描述符**中存在数据时，操作系统内核会将准备好的数据拷贝给应用程序并交回控制权。

**非阻塞I/O**：第一次发出http或者读写文件请求时，会向文件描述符读取数据，并且立即返回`EAGAIN` 错误，这表示文件描述符还在等待缓冲区的数据，也就是没准备好的意思，在第一次发出请求之后，应用程序会不断轮询调用这个请求，直到返回正确的值，此时就可以读取数据进行操作了。**为什么要这样做？**因为像第一种I/O模型，有阻塞状态，浪费了大量时间去等待，而非阻塞I/O能够立刻返回，在等待的过程中，去执行其他的操作，提高CPU利用率。

**文件描述符**：一个I/O操作，一个tcp连接一个websocket连接或者一个http请求的抽象的表示，用来区分这些请求，方便在唤醒这些事件时执行正确的操作。(**tips**：HTTP请求可能会存在复用文件描述符的情况，但是与之对应的还有`Stream ID`来区分它们，具体的步骤是这样的：**1.**网络轮询器检测到该文件描述符可读。**2.**用 `netpoll`读取文件描述符数据。**3.**发现该描述符属于http请求，使用http解析器解析`Sream ID`来区分不同的请求，并分发给goroutine)

**GO中的网络轮询器**：是一个全局的对象，将所有的文件描述符用一个链表来维护，可以实现快速的插入操作，至于查找操作，在发送I/O请求或者建立HTTP连接的时候(注册fd文件描述符)的时候，还会向epoll注册一份事件，此时也会保存对应fd节点的指针，当监听到返回的时候，则会返回这个指针，以此来实现快速查找。

**tips**：在网络轮询器初始化时，会涉及到一个pipe管道的初始化，还会创建一个epoll的文件描述符fd用于监听所有的事件，而在之后，在一定条件下会调用[`runtime.netpollBreak`](https://draveness.me/golang/tree/runtime.netpollBreak)方法，这个方法会通过这个管道发送中断等待某一事件的信号，然后让epoll去处理其他的事件，之前提到了向epoll注册事件，实际上他是如何实现的轮询操作的呢？事实上，在轮询的过程中，如果发现某一个事件没有准备好，则会调用gopark让出这个goroutine，陷入休眠状态，此后不会对其进行无用的轮询，从而减少CPU的损耗，而epoll负责监控事件，只有在事件发生时，才会通知GO程序然后来唤醒这个goroutine，同时，调用netpoll方法的时候，如果当前没有待处理的事件，就会陷入阻塞(这里不是睡眠！)，也就是epoll_wait，这可能会阻塞很久，如果休眠的时候有timer到期了怎么办？这个时候就会通过我们之前初始化的管道，调用`runtime.netpollBreak`方法发出中断信号，另外，当有goroutine需要执行，但是找不到M的时候，此时也会调用`runtime.netpollBreak`发出信号，让epoll_wait返回，来执行这个goroutine。

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







