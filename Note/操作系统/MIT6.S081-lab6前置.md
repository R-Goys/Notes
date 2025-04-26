#  MIT6.S081-lab5前置

这部分包含了设备中断和锁的内容

---

## 设备中断

之前系统调用的时候提过 usertrap ，而我们的设备中断，比如计时器中断也会在这里执行，我们可以看看具体的逻辑：

```c
void
usertrap(void)
{
  int which_dev = 0;

  if((r_sstatus() & SSTATUS_SPP) != 0)
	 ...
  } else if((which_dev = devintr()) != 0){
    // ok
  } else if((r_scause() == 15 || r_scause() == 13) && iscowpage(p->pagetable, r_stval())){
    ...
  } else {
	 ...
  }
  ...
  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2)
    yield();
  usertrapret();
}
```

我们可以发现，中间会为 which_dev 赋值，而当 which_dev 等于 2 的时候，我们会进入 yield 进行调度，而这里，就是我们实现时间片调度的关键地方，并且再devintr里面，如果识别为时钟中断，会将计时器自增，保证计时正确。

看了手册感觉云里雾里的，直接开读代码！

> [kernel/kernelvec.S](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/kernelvec.S), [kernel/plic.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/plic.c), [kernel/console.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/console.c), [kernel/uart.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/uart.c), [kernel/printf.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/printf.c)
>
> 虽然给了链接，但是最好是在本地读源码。

## kernel/console.c

先从简单的入手，根据手册我们可以知道， console.c 就是处理键盘和显示器的模块，下面开始读源码，我们会发现，有很多乱七八糟的函数，我们这一节需要学习一下中断，所以我们主要看看 `void consoleintr(int c)` 的调用即可，查看其引用，我们会发现，他最终会被 devintr 调用，当然，是在识别中断信号是由键盘发出的才会真的进行调用。

到这里，我们就可以将我们之前的知识串联起来，我们在输入字符的时候，会真正的产生键盘中断，并输入到显示屏上面。

按照顺序，先看看 `void consoleintr(int c)` 的调用者干了些什么

```c
// 处理 UART 中断，该中断可能由于有输入到达、
// 或 UART 准备好发送更多输出，或两者同时发生。
// 此函数由 devintr() 调用。
void
uartintr(void)
{
  // 循环读取并处理输入的字符。
  while(1){
    int c = uartgetc(); // 从 UART 读取一个字符
    if(c == -1)         // 没有更多输入时返回 -1
      break;
    consoleintr(c);     // 将字符传给 console 进行处理
  }

  acquire(&uart_tx_lock); // 加锁，防止并发修改发送缓冲区
  uartstart();            // 启动 UART 输出
  release(&uart_tx_lock);
}
```

uartintr 实际上是直接有 devintr 调用的，他上层的调用我们已经很清楚了，所以不需要再看上层干了什么，我们专注于下层的实现。我们可以看见，uartintr 循环从 uart 里面读取字符，然后传给我们刚刚提到的 consoleintr 处理，最终发送给缓冲区，等待输出，我们可以进一步看看如何从 UART 中读取字符：

```c
// 从 UART 读取一个输入字符。
// 如果没有可读的字符，则返回 -1。
int
uartgetc(void)
{
  // 检查接收状态寄存器（LSR）的最低位是否为 1，
  // 表示是否有输入数据准备好。
  if(ReadReg(LSR) & 0x01){
    // 如果有输入数据，读取接收保持寄存器（RHR）中的字符返回。
    return ReadReg(RHR);
  } else {
    // 否则返回 -1，表示没有数据可以读取。
    return -1;
  }
}
```

ReadReg 就不再继续深入了，他这里实现的实际上就是从寄存器 RHR 读取一个字符，并且每次读取都会以一种类似队列的模式踢出读取的字符，把下一个字符放到 RHR 中，如果我们看他的源码，会发现没有这一部分的逻辑，这是由硬件实现的。

回到上一层，我们可以看见我们调用了 consoleintr 去处理这一个字符，但是我们还没有去看过他的源码，这里就去看一下~

```c
void
consoleintr(int c)
{
  acquire(&cons.lock); // 获取锁，防止并发修改 cons 结构体。

  switch(c){
  case C('P'):  // Ctrl+P：打印当前进程列表。
    procdump();
    break;
    
  case C('U'):  // Ctrl+U：清除当前输入行（kill line）。
    // cons.e 指的是当前写入位置的下标索引。
    while(cons.e != cons.w &&
          cons.buf[(cons.e-1) % INPUT_BUF_SIZE] != '\n'){
      cons.e--;                // 索引--
      consputc(BACKSPACE);     // 删除并移动光标
    }
    break;

  case C('H'): // Ctrl+H，表示退格（Backspace）
  case '\x7f': // Delete 键的 ASCII（127），功能与退格相同
    if(cons.e != cons.w){
      cons.e--;                // 删除一个字符
      consputc(BACKSPACE);     // 回显退格符
    }
    break;

  default:
    // 只在字符非 0 且缓冲区未满时处理
    if(c != 0 && cons.e-cons.r < INPUT_BUF_SIZE){
      // 将回车符转为换行符，统一处理
      c = (c == '\r') ? '\n' : c;

      consputc(c);  // 将字符回显到控制台

      // 将字符写入缓冲区 cons.buf 中
      cons.buf[cons.e++ % INPUT_BUF_SIZE] = c;

      // 如果收到换行、Ctrl+D（EOF），或缓冲区已满，则唤醒 consoleread
      if(c == '\n' || c == C('D') || cons.e-cons.r == INPUT_BUF_SIZE){
        cons.w = cons.e;      // 更新写指针，表示有一行可读
        wakeup(&cons.r);      // 唤醒等待读取的 consoleread()
      }
    }
    break;
  }

  release(&cons.lock); // 释放锁
}
```

我们发现，有很多快捷键可以帮助我们识别一些指令，比如清空行，打印进程，我们可以去试试，这些确实是可以执行的!

当 UART 接收到一个字符时，会调用这个函数处理输入。主要功能是处理用户的输入字符，包括特殊控制字符（如 Ctrl+U 清空行、Ctrl+P 打印进程），并将处理后的字符追加到控制台缓冲区 cons.buf 中。如果接收到换行符（表示一行输入完成），则唤醒等待输入的 consoleread()。 

我们大体梳理了一下这个函数的逻辑，我们可以仔细看看其中用到的函数，先看看第一个 procdump ：

```go
void
procdump(void)
{
  // 进程状态字符串表，对应 enum procstate 的枚举值。
  static char *states[] = {
  [UNUSED]    "unused",  // 未使用
  [USED]      "used",    // 已分配但未就绪
  [SLEEPING]  "sleep ",  // 睡眠中
  [RUNNABLE]  "runble",  // 可运行
  [RUNNING]   "run   ",  // 正在运行
  [ZOMBIE]    "zombie"   // 僵尸态（已退出但未被回收）
  };

  struct proc *p;
  char *state;

  printf("\n"); // 输出一个空行用于分隔

  // 遍历所有进程表项
  for(p = proc; p < &proc[NPROC]; p++){
    if(p->state == UNUSED)  // 跳过未使用的进程项
      continue;

    // 获取进程的状态对应的字符串
    if(p->state >= 0 && p->state < NELEM(states) && states[p->state])
      state = states[p->state];
    else
      state = "???";  // 状态非法或未知

    // 打印进程的 pid、状态和进程名
    printf("%d %s %s", p->pid, state, p->name);
    printf("\n");
  }
}
```

这里就是执行了打印进程的逻辑，不是重点，next。

而当我们在键盘上按 Ctrl + U 的时候，则会清空我们当前输入的行，我们可以看看这个函数 consputc ：

```c
//
// consputc - 向 UART 发送一个字符。
// 该函数由 printf() 调用，也用于回显输入字符；但不会由 write() 调用。
// 
void
consputc(int c)
{
  if(c == BACKSPACE){
    // 如果用户输入了退格（Backspace），用空格覆盖原字符。
    uartputc_sync('\b'); // 发送退格符（移动光标到前一个位置）
    uartputc_sync(' ');  // 发送空格覆盖原字符
    uartputc_sync('\b'); // 发送退格符，将光标移回原位置
  } else {
    // 向 UART 发送普通字符
    uartputc_sync(c);
  }
}
```

我们可以看见，这个函数可以移动我们的光标，以及向 UART 发送我们的字符。在 uartputc_sync 中，最终会执行我们的  WriteReg 来向指定的寄存器写入字符。之后，就由硬件来完成字符的显示工作，这样，就完成了一个字符的清空退格和显示。

事实上，输入的各种字符最终都会通过这个函数来显示出来。

而在输入换行，或者超出缓冲区的时候，会调用wakeup触发调度，来调用 consoleread 从而读取控制台的信息，但是为什么会知道wakeup就会调用这个 consoleread ？：

```c
//
// 用户态的 read() 系统调用读取控制台时，会调用这里。
// 作用是：从 cons.buf 中拷贝一整行（或者尽可能多的输入）到 dst 指向的内存。
// user_dst 参数标记 dst 是用户空间地址还是内核空间地址。
//
int
consoleread(int user_dst, uint64 dst, int n)
{
  uint target;
  int c;
  char cbuf;

  target = n; // 保存最初要求读取的字节数
  acquire(&cons.lock); // 加锁，保护 cons 共享数据

  while(n > 0){
    // 如果当前没有可读的数据，进入睡眠，等待输入。
    while(cons.r == cons.w){
      if(killed(myproc())){
        // 如果当前进程已经被杀死，直接返回错误。
        release(&cons.lock);
        return -1;
      }
      // 这里传入了一个 cons.r 的参数，这样我们就可以正确唤醒了！
      sleep(&cons.r, &cons.lock); // 没数据可读时挂起等待唤醒
    }

    // 从输入缓冲区读取一个字符
    c = cons.buf[cons.r++ % INPUT_BUF_SIZE];

    if(c == C('D')){  // 遇到 Ctrl+D，表示文件结束（EOF）
      if(n < target){
        // 如果已经读入了一些数据，把 ^D 留在缓冲区，等下次再处理
        cons.r--;
      }
      break; // 退出读取循环
    }

    // 把读到的一个字节拷贝到用户空间（或内核空间）dst指向的地方
    cbuf = c;
    // 典中典之 copyout
    if(either_copyout(user_dst, dst, &cbuf, 1) == -1)
      break; // 如果拷贝失败，直接结束读取

    dst++; // dst 指针后移
    --n;   // 剩余要读取的字节数减一

    if(c == '\n'){
      // 如果读到了换行符，说明一整行已经读完了
      break;
    }
  }

  release(&cons.lock); // 释放锁

  return target - n; // 返回实际读到的字节数
}
```

那么问题来了，我们在通过 wakeup 是进入到了哪一个进程？哪一个进程为我们准备好了 read ？很简单，我们知道，我们所有的输入基本都是在 shell 里面执行的，所以我们可以回到 user/sh.c 中，去查看，很轻松可以找到，里面的调用链路：

> main -> getcmd -> gets -> read

以此来读取我们输入的字符，而我们在shell里面等待的时候，这个read是处于睡眠的状态，通过之前提过的快捷键可以知道我们目前运行的进程：

```bash
$ 
1 sleep  init
2 sleep  sh
```

sh处于睡眠，而一旦我们敲击键盘，输入字符的那一瞬间，会产生中断，并且将字符输入缓冲区，一旦我们输入换行符，或者缓冲区溢出了，就会唤醒 consoleread 来读取我们的缓冲区（控制台）数据，随后，通过shell去识别这行内容，并执行相关程序。这样，就形成了一次输入的闭环。

## user/printf.c

回到另一个关于屏幕显示的函数，printf，它实际上没有很复杂，除了一堆字符串判断逻辑之外，就仅仅是系统调用 write 一个字符到屏幕上，忽略获取参数的部分，我们可以进入 write 这个系统调用里面详细看看他干了什么：

```c
// 向文件 f 写数据。
// addr 是用户空间的虚拟地址。
int
filewrite(struct file *f, uint64 addr, int n)
{
  int r, ret = 0;

  // 如果文件不可写，直接返回错误。
  if(f->writable == 0)
    return -1;

  if(f->type == FD_PIPE){
    // 如果是管道类型文件，调用管道的写操作。
    ret = pipewrite(f->pipe, addr, n);

  } else if(f->type == FD_DEVICE){
    // 如果是设备文件，调用对应设备的写操作。
    if(f->major < 0 || f->major >= NDEV || !devsw[f->major].write)
      return -1;
    ret = devsw[f->major].write(1, addr, n);  // 调用设备的 write 函数
    //这里是我们的重点

  } else if(f->type == FD_INODE){
    // 如果是普通文件（inode文件），进行文件系统写操作。

    // 为了避免超出日志的最大事务大小（MAXOPBLOCKS），
    // 每次最多只写 max 字节。
    int max = ((MAXOPBLOCKS-1-1-2) / 2) * BSIZE;
    int i = 0;
    while(i < n){
      int n1 = n - i;
      if(n1 > max)
        n1 = max;

      begin_op();        // 开始一个文件系统事务
      ilock(f->ip);       // 加锁 i-node
      if ((r = writei(f->ip, 1, addr + i, f->off, n1)) > 0)
        f->off += r;     // 更新文件偏移量
      iunlock(f->ip);     // 解锁 i-node
      end_op();          // 结束事务

      if(r != n1){
        // 如果实际写入的字节数与预期不同，说明出错了，停止写入。
        break;
      }
      i += r; // 写入成功，继续写下一部分
    }
    ret = (i == n ? n : -1);  // 如果全部写完返回写入的字节数，否则返回错误

  } else {
    // 如果文件类型不是以上几种，说明出错，触发 panic。
    panic("filewrite");
  }

  return ret;
}
```

姑且给出全部注释，但是对于本章无足轻重的部分就不讲解了，重点看看我们的 `ret = devsw[f->major].write(1, addr, n);` 的调用，它是什么意思？什么时候会进入到他的里面？

在进入这里之前，我们会判断传入的文件描述符指向的文件是一个设备，并且存在对应的 write 函数，这样，我们可以专门的去访问这个write，比如说我们的 console 设备就会提前注册一个 write 的函数，这个函数会将对应的字符 write 到指定的文件中，在这里，就是终端，我们注册函数是这个：

```c
int consolewrite(int user_src, uint64 src, int n)
{
  int i;

  // 遍历需要写入的字节数。
  for(i = 0; i < n; i++){
    char c;
    
    // 从用户空间复制一个字节到变量 c 中。
    // either_copyin() 用于检查地址有效性，并将数据从用户空间复制到内核空间。
    if(either_copyin(&c, user_src, src+i, 1) == -1)
      break;  // 如果复制失败，则退出，返回已写入的字节数。

    // 调用 uartputc() 将字符 c 发送到 UART，用于输出到控制台。
    uartputc(c);
  }

  // 返回实际写入的字节数。
  return i;
}
```

我们的 uartputc 会将字符存入 UART 的缓冲区，在之后，会由硬件输出到控制台：

```c
// 将字符添加到 UART 发送缓冲区，并在缓冲区未满时启动 UART 传输。
// 如果缓冲区已满，则会阻塞直到有空间可用。
// 该函数不适合在中断中调用，适合用在写操作中。
// 用于将字符通过 UART 输出到控制台。
void uartputc(int c) {
  // 获取 UART 发送缓冲区的锁，确保线程安全。
  acquire(&uart_tx_lock);

  // 如果系统处于 panic 状态，进入死循环，阻止继续执行。
  if (panicked) {
    for (;;) 
      ; // 系统处于 panic 状态，不再继续执行。
  }

  // 如果 UART 发送缓冲区满，等待直到有空间可用。
  while (uart_tx_w == uart_tx_r + UART_TX_BUF_SIZE) {
    // 缓冲区已满，等待 uartstart() 函数发送字符并腾出空间。
    sleep(&uart_tx_r, &uart_tx_lock);
  }

  // 将字符 c 放入 UART 发送缓冲区。
  uart_tx_buf[uart_tx_w % UART_TX_BUF_SIZE] = c;
  uart_tx_w += 1; // 更新写指针。

  // 调用 uartstart() 启动 UART 传输，开始将缓冲区中的字符发送出去。
  uartstart();

  // 释放 UART 发送缓冲区的锁。
  release(&uart_tx_lock);
}
```

总结一下，最终的 write 会调用 uartputc 这个函数，然后这里就会向终端的缓冲区 uart_tx_buf 写入数据，如果缓冲区满了，就会休眠等待，中途如果被唤醒，写入数据，然后就会调用 uartstart 向终端显示数据了，为什么会有缓冲区，为什么需要睡眠？操作系统是个天生的多线程并发运行的，应用程序可能一口气产生大量数据，但 UART 硬件发送速度慢，仅有一个进程可能感受不到什么，但是如果进程很多，缓冲区可能在一段时间非常庞大，占用了大量的内存就不可想象了！所以这里陷入睡眠等待是比较好的选择。

这里的具体的逻辑其实和之前的键盘中断是类似的，而之前 console.c 的时候，也提到过 uartstart，但是没有分析，所以在这里分析一下他干了什么：

```c
// 如果 UART 处于空闲状态，并且传输缓冲区中有字符等待发送，则发送字符。
// 调用者必须持有 uart_tx_lock。
// 该函数同时可以从中断的顶部（中断处理程序）和底部（常规代码）调用。
void
uartstart()
{
  while(1){
    // 如果传输缓冲区为空，表示没有数据等待发送。
    if(uart_tx_w == uart_tx_r){
      ReadReg(ISR); // 读取中断状态寄存器，清除相关中断标志。
      return; // 返回，不需要发送任何数据。
    }
    
    // 如果 UART 的传输保持寄存器未空闲，表示无法再发送数据。
    if((ReadReg(LSR) & LSR_TX_IDLE) == 0){
      // 传输保持寄存器已满，无法发送新的字节。
      // 当寄存器空闲时会触发中断，我们就能继续发送下一个字节。
      return;
    }
    
    // 从缓冲区中取出一个待发送的字符。
    int c = uart_tx_buf[uart_tx_r % UART_TX_BUF_SIZE];
    uart_tx_r += 1; // 更新缓冲区的读取指针。
    
    // 可能有调用 uartputc() 的地方在等待缓冲区空间的释放，唤醒它。
    wakeup(&uart_tx_r);
    
    // 向 UART 传输保持寄存器写入字符。
    WriteReg(THR, c);
  }
}

```

这里我其实没太理解，因为在 consoleintr 里面已经 Write 了一遍，为什么这里还需要再执行一遍？在这里先保留一下这个问题吧。

回来解决这个疑惑，我们在 consoleintr 会将字符显示到屏幕上，这部分讲义其实也没讲清楚，我们可以回归手册来深入理解这部分，我们 consoleintr 会将数据显示到屏幕上并写入到 input 缓冲区，也会对一些特殊键进行处理，当然，如果我们遇到换行符，就会将控制权交给 consoleread 他则是我们的 read 系统调用，此时，会将这一行内容复制到用户空间。

同时，为什么会有 uartstart 这个东西？实际上，在这里是排不上用场的，因为此时已经把数据给读取完了，那么什么时候会派上用场？当我们执行 write 系统调用的时候，最终会到达 uartputc ，就跟我们之前介绍 printf 提过的， 会存在一个 uart_tx_buf 缓冲区，这个缓冲区可以实现异步的写入和读取，而我们的 uartstart 恰恰就会去读取这一串缓冲区，并写入到指定的寄存器上，而当我们的 UART 发送完一个字节，就会产生一个中断，从而再次调用这个 uartstart ，来检查是否真的完成了发送，并且将下一个缓冲的输出字符交给寄存器，这样就实现了解耦。

## kernel/plic.c

这玩意是啥？有啥用？

举个例子，如果同时有多个中断程序需要我们去处理，比如键盘中断，磁盘 I/O ，我们就需要去设定一个优先级，并且能够识别具有最优优先级的中断程序，从而去处理，实现这一组的代码，则是我们的 plic.c ， 在操作系统初始化的时候，会调用：

```c
void
plicinit(void)
{
  // 设置外设中断（IRQ）的优先级，使能中断。
  // 如果优先级为 0，表示禁用；非零表示启用并设定优先级。
  
  // 将 UART0 的中断优先级设置为 1（使能 UART0 中断）
  *(uint32*)(PLIC + UART0_IRQ*4) = 1;

  // 将 VIRTIO0（虚拟磁盘设备）的中断优先级设置为 1（使能 VIRTIO0 中断）
  *(uint32*)(PLIC + VIRTIO0_IRQ*4) = 1;
}
```

这里都将优先级设置为 1 了，这两个都能够被判断了，但是两者优先级上却没什么区别，这里就会用到调度策略（Ai说的）

同时，我们可以在 devintr 里面发现， `plic_claim` 这个函数，它可以帮助我们获取当前优先级最高的中断：

```c
int
plic_claim(void)
{
  int hart = cpuid();
  int irq = *(uint32*)PLIC_SCLAIM(hart);  // 从 PLIC 中读取当前需要处理的中断号
  return irq;
}
```

很简单，这里的 `PLIC_SCLAIM` 是直接从内存中获取数据，不再赘述，简单来说，就是在内存中的指定位置设定优先级之后，硬件会帮你判断。

随后，我们就可以根据返回的中断号来执行指定的中断程序了。

除此之外， kernelvec 和 uservec 差不多，就不多介绍了。

## lecture部分

中断和系统调用很相似，虽然都使用了相同的保存状态，恢复状态的机制，并且在一个地方处理，但是他们并不一样，其实大部分内容都已经在源码阅读那一块讲完了。。哈哈。**但是**依旧有一点是值得提及的：

**并发**，虽然现在还没有到并发的部分，但是这里依旧是涉及到一定的并发的，比如说我们的设备和 cpu ：当 UART 向 Console发送字符的时候， CPU 会被唤醒，返回执行 shell ，而他可能会执行再执行一次系统调用，向 buffer 中写入一个字符，而这是在并行的执行。

这样的并发叫做 Consumer/Producer 并发，在 buffer 的例子里面，我们的 producer 可以一直写入数据， 使得写指针 + 1 ，而 buffer 满的时候， 写指针则需要停下，我们的 uartintr 就会从读指针中读取一个字符，再通过 UART 发送，令读指针 + 1 ，如果读指针追上了写指针，则说明 buffer 为空了。而这段 buffer 在内存中仅此一份。 

另外随便提一下，过去的中断处理还是很快的，原因是过去的计算机很简单，中断处理也不复杂，现在设备变得很复杂，处理器需要干的事情变得更多了，就慢了，所以产生中断之前，设备就会做大量的操作，减轻 cpu 的负担。

同时，如果有一个高性能设备，如网卡，不断地接收到大量的包，产生的中断就会很多，这里的解决办法就是使用轮询，但是这里的轮询仅仅是设备 I/O 轮询，但是对于中断触发少的设备，我们依旧采取中断的方式，这样，对于中断频繁的设备，我们 cpu 主动去检查是否有数据到来，这样，来提高性能。
