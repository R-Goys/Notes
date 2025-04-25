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

uartintr 实际上是直接有 devintr 调用的，他上层的调用我们已经很清楚了，所以不需要再看上层干了什么，我们专注于下层的实现。我们可以看见，uartintr 循环从 uart 里面读取字符，然后传给我们刚刚提到的consoleintr处理，最终发送给缓冲区，等待输出，我们可以进一步看看如何从 UART 中读取字符：

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

ReadReg就不再继续深入了，他这里实现的实际上就是从寄存器 RHR 读取一个字符，并且每次读取都会以一种类似队列的模式踢出读取的字符，把下一个字符放到 RHR 中，如果我们看他的源码，会发现没有这一部分的逻辑，这是由硬件实现的。

回到上一层，我们可以看见我们调用了consoleintr去处理这一个字符，但是我们还没有去看过他的源码，这里就去看一下~
