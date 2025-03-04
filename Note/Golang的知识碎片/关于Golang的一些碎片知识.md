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

**最佳实践**：

-  合理利用 `context.WithCancel` 取消 Goroutine，避免泄露，减少资源占用
-  使用 `context.WithTimeout` 限制任务时间，防止超时阻塞
-  使用 `context.WithDeadline` 确保任务在固定时间前完成
-  使用 `context.WithValue` 传递请求作用域数据(这个不常用)
-  `context` 只应该作为参数传递，而不是结构体字段，更不是全局变量

----

## 3. 计时器

表面上是切片，内部使用四叉堆维护timer(计时器)，堆顶元素是距离截止时间最近的计时器，四叉堆，顾名思义，就是每个节点只能有四个子节点，Go语言通过网络轮询器监听最近到期的timer是否到期，如果到期，则会执行回调。网络轮询器实际上还负责I/O事件和网络连接，如果这三者同时发生，timer执行的优先度是最低的。

**最佳实践**：

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

**文件描述符**：表示一个I/O操作，一个tcp连接，一个websocket连接，一个http请求的抽象的表示，用来区分这些请求，方便再回调时执行正确的操作。(**tips**：HTTP请求可能会存在复用文件描述符的情况，但是与之对应的还有`Stream ID`来区分它们，具体的步骤是这样的：**1.**网络轮询器检测到该文件描述符可读。**2.**用 `netpoll`读取文件描述符数据。**3.**发现该描述符属于http请求，使用http解析器解析`Sream ID`来区分不同的请求，并分发给goroutine)

**GO中的网络轮询器**：是一个全局的对象，将所有的文件描述符用一个链表来维护，可以实现快速的插入操作，至于查找操作，在发送I/O请求或者建立HTTP连接的时候(注册fd文件描述符)的时候，还会向epoll注册一份事件，此时也会保存对应fd节点的指针，当监听到返回的时候，则会返回这个指针，以此来实现快速查找。

**tips**：在网络轮询器初始化时，会涉及到一个pipe管道的初始化，还会创建一个epoll的文件描述符fd用于监听所有的事件，而在之后，在一定条件下会调用[`runtime.netpollBreak`](https://draveness.me/golang/tree/runtime.netpollBreak)方法，这个方法会通过这个管道发送中断等待某一事件的信号，然后让epoll去处理其他的事件，之前提到了向epoll注册事件，实际上他是如何实现的轮询操作的呢？事实上，是操作系统在负责监控，只有在事件发生时，才会通知Go程序(这方面是问的AI，详情我也不清楚)
