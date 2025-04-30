# MIT6.S081-lab8前置

注：本部分除了文件还包含了调度的内容。

## 调度

调度涉及到保存寄存器，恢复寄存器，就这一点而言，和我们的 trap 很像，但是实际上，我们实现并不是复用了 trap 的逻辑，我们可以在 kernel/swtch.S 里面找到关于上下文的保存与恢复的代码。

它是由什么调用的？调用链路如下：

> usertrap -> yield -> sched -> swtch

我们可以在 proc.h 中，找到关于我们保存的 context 结构体的痕迹：

```c
// 用于内核线程上下文切换时保存寄存器的结构体。
struct context {
  uint64 ra;   // return address，返回地址寄存器（保存 PC），用于从 swtch 返回
  uint64 sp;   // 栈指针，指向该线程的内核栈顶

  // 以下是 callee-saved 寄存器，根据 RISC-V 调用约定需要在函数调用之间保持不变
  uint64 s0;
  uint64 s1;
  uint64 s2;
  uint64 s3;
  uint64 s4;
  uint64 s5;
  uint64 s6;
  uint64 s7;
  uint64 s8;
  uint64 s9;
  uint64 s10;
  uint64 s11;
};
```

由此我们可以看出，我们保存了一些临时值/栈指针以及返回的地址，以保证我们能够实现正确的上下文切换，返回。

但是这样就能切换了？我们可以来看看 sched 的代码，看看他干了些什么：

```c
void
sched(void)
{
  int intena;
  struct proc *p = myproc(); // 获取当前运行的进程指针

  // 下面是一系列 check，确保调用 sched 时的前提条件正确：

  // 必须持有当前进程的锁，否则可能导致竞态
  if(!holding(&p->lock))
    panic("sched p->lock");

  // noff == 1 表示当前只屏蔽了一层中断（通过 push_off()），是内核中强约束要求
  if(mycpu()->noff != 1)
    panic("sched locks");

  // 当前进程不应该是 RUNNING 状态，调用 sched 前必须先设置为 SLEEPING、RUNNABLE 等
  if(p->state == RUNNING)
    panic("sched running");

  // 中断应该已经被关闭，避免中断过程中切换上下文
  if(intr_get())
    panic("sched interruptible");

  // 保存当前中断开启状态，稍后恢复
  intena = mycpu()->intena;

  // 切换上下文：保存当前进程的 context，跳转到 CPU 调度器 context（在 scheduler 中）
  swtch(&p->context, &mycpu()->context);

  // 恢复中断开启状态
  mycpu()->intena = intena;
}
```

此时我们的 swtch 通过 mycpu 切换到另一个线程，这里的切换的线程是由谁决定的？这里需要回到我们 kernel 的启动主函数：

```c
void
main()
{
  // 只有引导 cpu 才会进行初始化操作系统
  if(cpuid() == 0){
    consoleinit();
    printfinit();
    printf("\n");
    printf("xv6 kernel is booting\n");
    printf("\n");
    kinit();         // physical page allocator
    kvminit();       // create kernel page table
    kvminithart();   // turn on paging
    procinit();      // process table
    trapinit();      // trap vectors
    trapinithart();  // install kernel trap vector
    plicinit();      // set up interrupt controller
    plicinithart();  // ask PLIC for device interrupts
    binit();         // buffer cache
    iinit();         // inode table
    fileinit();      // file table
    virtio_disk_init(); // emulated hard disk
    userinit();      // first user process
    __sync_synchronize();
    started = 1;
  } else {
    while(started == 0)
      ;
    __sync_synchronize();
    printf("hart %d starting\n", cpuid());
    kvminithart();    // turn on paging
    trapinithart();   // install kernel trap vector
    plicinithart();   // ask PLIC for device interrupts
  }
  // 最后所有的 cpu 都会执行调度器
  scheduler();        
}
```

我们可以发现，在 main 函数的末尾，会调用一个 scheduler ，也就是调度器，这个函数会无限循环，帮助整个操作系统去调度线程，所以我们这部分的核心就是研究 scheduler 是如何工作的：

```c
void
scheduler(void)
{
  struct proc *p;
  struct cpu *c = mycpu();  // 获取当前 CPU 的结构体

  c->proc = 0;  // 当前 CPU 还没有运行任何进程
  for(;;){      // 调度器是一个无限循环，不会退出
    // 上一个运行的进程可能关闭了中断，为了防止所有进程都阻塞导致死锁，这里强制打开中断
    intr_on();

    int nproc = 0;  // 当前活跃（非 UNUSED）的进程数量
    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);  // 加锁，防止状态竞争

      if(p->state != UNUSED) {
        nproc++;  // 统计当前系统中还存在的进程数
      }

      if(p->state == RUNNABLE) {  // 找到一个可以运行的进程
        p->state = RUNNING;       // 将状态设置为正在运行
        c->proc = p;              // 标记当前 CPU 正在运行该进程

        // 切换上下文：保存当前 CPU 的 scheduler 上下文，跳转到进程的上下文继续执行
        swtch(&c->context, &p->context);

        // 一旦 swtch 返回，说明该进程让出了 CPU，调度器重新获得控制权
        c->proc = 0;  // 清除当前 CPU 上运行的进程指针
      }

      release(&p->lock);  // 解锁
    }
	// 如果系统中只剩 init 和 shell，没有用户进程
    if(nproc <= 2) {    // 调用 wfi（等待中断），进入低功耗状态，直到有中断发生再唤醒
      intr_on();
#ifndef LAB_FS
      asm volatile("wfi");
#endif
    }
  }
}
```

我们可以看到，我们其实最开始所有的 cpu 都运行在 scheduler 中，一旦发现需要运行的进程，就会启动调度循环，通过 swtch 切换到需要被调度的进程。而这里和 yield 是相辅相成的，当我们的进程运行时间耗尽或者执行完毕，就会在 yield 中执行 swtch 回到我们的调度循环，进而执行其他进程的调度或者陷入休眠，这里也是一个理解起来很有意思的点。

我们最开始完成了一系列启动程序之后（包括 shell），就会进入到调度程序中，如果进程中只有 shell 和 init 进程，那么所有的 cpu 没有什么可以调度的，就会陷入低功耗模式，一旦我们的 shell 通过 fork 创建了一个进程，这个 fork 也会将这个进程注册到全局的进程数组中，之后会执行 exec ，将我们需要的程序加载进去，这样，我们的 scheduler 就可以正常的去调度这个进程了！

## 休眠和唤醒

我们之前在学习中断的时候曾看见过 sleep 和 wakeup 这样的函数，他们实际上也和调度有着千丝万缕的关系，我们在等待事件的时候，会希望它能够不消耗 cpu 资源，而能够让其他的进程去占用这个 cpu ，于是，我们就引入了 sleep 和 wakeup 这两个函数，通过他们，我们可以让进程在等待时陷入睡眠，而当我们未来有一个进程完成了这个进程所需要的时间，我们就可以通过 wakeup去唤醒它，在 xv6 中，有这样一个简单的休眠和唤醒的机制：

```c
// 原子地释放锁并在 chan 上睡眠，chan 可以理解为进程陷入休眠的理由
// 唤醒时会重新获取锁。
void
sleep(void *chan, struct spinlock *lk)
{
  struct proc *p = myproc();
  
  // 必须获取 p->lock 来更改 p->state，然后调用 sched。
  // 一旦我们持有 p->lock，我们可以保证不会错过任何的 wakeup
  // （因为wakeup 会锁定 p->lock），
  acquire(&p->lock);
  
  // 在进入睡眠之前释放原始锁，允许其他进程运行。
  release(lk);

  // 将进程的状态设置为睡眠，并将其与提供的通道关联。
  p->chan = chan;
  p->state = SLEEPING;

  // 切换到调度器，让给其他进程运行
  sched();

  // 唤醒后进行清理。
  // 清除进程的通道，表示进程不再在该通道上睡眠。
  p->chan = 0;

  // 唤醒后重新获取原始的锁。
  release(&p->lock);
  acquire(lk);
}

// 唤醒所有在 chan 上睡眠的进程。
// 必须在没有任何 p->lock 的情况下调用。
void
wakeup(void *chan)
{
  struct proc *p;

  // 遍历进程表中的所有进程。
  for(p = proc; p < &proc[NPROC]; p++) {
    // 因为当前进程一定是在运行的，所以跳过当前进程（myproc）。
    if(p != myproc()){
      acquire(&p->lock);  // 获取锁，以安全地修改进程状态。
      
      // 如果处于睡眠且与给定的休眠原因一样，则唤醒。
      if(p->state == SLEEPING && p->chan == chan) {
        // 将进程状态更改为可运行（RUNNABLE），使其可以被调度。
        p->state = RUNNABLE;
      }
      // 在修改进程状态后释放锁。
      release(&p->lock);
    }
  }
}
```

大部分内容都已经在注释中标好了，但是仍然值得一提的是 chan， chan 指的就是进程陷入休眠的原因，我们如果想要因为一个操作而唤醒多个进程，就可以使用将这多个进程陷入休眠的原因写成一样的。

通过 sleep 和 wakeup 的机制，我们也可以实现信号量的机制。

另外，xv6 中有一个利用了这个机制实现了比较好的例子是 pipe ，当我们像管道中写入数据的时候，我们会尝试获取这个管道的锁，获取成功，则会开始下一步判断，如果管道缓冲区已经满了，就会唤醒读管道进行读取，然后令当前进程休眠从而释放锁，其实也不算很复杂，具体的逻辑如下：

```c
// 向管道中写数据。
// 参数：pi - 管道，addr - 写入数据的地址，n - 要写入的数据字节数
int
pipewrite(struct pipe *pi, uint64 addr, int n)
{
  int i = 0;
  struct proc *pr = myproc();  // 获取当前进程

  acquire(&pi->lock);  // 获取管道的锁，确保对管道的访问是互斥的
  while(i < n){  // 循环写入数据
    if(pi->readopen == 0 || killed(pr)){  // 如果管道的读端已关闭或者进程已被杀死
      release(&pi->lock);  // 释放锁
      return -1;  // 写入失败
    }
    // 如果管道已满（即写指针与读指针的差值等于管道大小），则进入休眠
    if(pi->nwrite == pi->nread + PIPESIZE){
      wakeup(&pi->nread);  // 唤醒可能在管道上等待的读取进程
      sleep(&pi->nwrite, &pi->lock);  // 进入休眠，等待管道有空闲空间
    } else {
      char ch;
      // 从进程的地址空间读取一个字节的数据
      if(copyin(pr->pagetable, &ch, addr + i, 1) == -1)
        break;  // 如果读取失败，退出
      pi->data[pi->nwrite++ % PIPESIZE] = ch;  // 将数据写入管道并更新写指针
      i++;  // 增加已写入字节数
    }
  }
  wakeup(&pi->nread);  // 唤醒可能在管道上等待的读取进程
  release(&pi->lock);  // 释放锁

  return i;  // 返回写入的字节数
}

// 从管道中读取数据。
// 参数：pi - 管道，addr - 读取数据的地址，n - 要读取的数据字节数
int
piperead(struct pipe *pi, uint64 addr, int n)
{
  int i;
  struct proc *pr = myproc();  // 获取当前进程
  char ch;

  acquire(&pi->lock);  // 获取管道的锁，确保对管道的访问是互斥的
  // 如果管道为空且写端没有关闭，进入休眠等待写入数据
  while(pi->nread == pi->nwrite && pi->writeopen){
    if(killed(pr)){  // 如果进程被杀死，退出
      release(&pi->lock);  // 释放锁
      return -1;  // 读取失败
    }
    sleep(&pi->nread, &pi->lock);  // 进入休眠，等待数据写入
  }
  for(i = 0; i < n; i++){
    // 如果管道为空，停止读取
    if(pi->nread == pi->nwrite)
      break;
    ch = pi->data[pi->nread++ % PIPESIZE];  // 从管道读取一个字节数据
    // 将数据写入进程的地址空间
    if(copyout(pr->pagetable, addr + i, &ch, 1) == -1)
      break;  // 如果写入失败，退出
  }
  wakeup(&pi->nwrite);  // 唤醒可能在管道上等待写入的进程
  release(&pi->lock);  // 释放锁
  return i;  // 返回读取的字节数
}
```

## 进程

关于进程，我们还需要介绍一些东西， wait/exit/kill 这三个调用。

我们在写第一个 lab 的时候，常常可以看见 exit 和 wait 这两个，所以从这两个开始， exit 是什么？就是令当前的进程正常退出，这个进程在执行 exit 过程中会变成僵尸状态，此时，他将无法被再次调度。

但是值得注意的是，我们在 exit 的之后，当前进程会被标记为僵尸进程，而不是 UNUSED ，我们的 fork 系统调用无法复用僵尸进程占用的空间，通常，我们的父进程执行 wait 的时候，我们的子进程会被正确回收。

```c
// 退出当前进程。该函数不会返回。
// 退出的进程会进入僵尸状态，
// 直到其父进程调用 wait()。
void
exit(int status)
{
  struct proc *p = myproc();  // 获取当前进程

  if(p == initproc)  // 如果当前进程是初始化进程，触发 panic，因为 init 进程不能退出
    panic("init exiting");

  // 关闭所有打开的文件。
  for(int fd = 0; fd < NOFILE; fd++){
    if(p->ofile[fd]){  // 如果该文件描述符有打开的文件
      struct file *f = p->ofile[fd];
      fileclose(f);  // 关闭文件
      p->ofile[fd] = 0;  // 将文件描述符置为 0，表示关闭
    }
  }

  // 清理当前进程的工作目录。
  begin_op();
  iput(p->cwd);  // 释放工作目录的 inode
  end_op();
  p->cwd = 0;  // 清除工作目录指针

  acquire(&wait_lock);  // 获取等待锁，确保进程退出时不会有竞争条件

  // 将当前进程的子进程交给 init 进程管理。
  reparent(p);

  // 父进程可能正在等待子进程退出，唤醒父进程。
  wakeup(p->parent);

  acquire(&p->lock);  // 获取当前进程的锁

  // 设置当前进程的退出状态，并将其标记为僵尸状态。
  p->xstate = status;  // 设置退出状态
  p->state = ZOMBIE;  // 将进程状态设置为 ZOMBIE

  release(&wait_lock);  // 释放等待锁

  // 跳转到调度器，永远不返回。
  sched();

  // 如果调度器返回，表示发生了错误，触发 panic。
  panic("zombie exit");
}
```

我们上面提到了父进程执行 wait 等待子进程退出，此时，所谓的僵尸进程的空间才会被真正释放：

```c
// 等待一个子进程退出并返回其 pid。
// 如果该进程没有子进程，则返回 -1。
int
wait(uint64 addr)
{
  struct proc *pp;
  int havekids, pid;
  struct proc *p = myproc();  // 获取当前进程

  acquire(&wait_lock);  // 获取等待锁，确保对进程表的访问是互斥的

  for(;;){
    // 扫描进程表，查找已退出的子进程。
    havekids = 0;
    for(pp = proc; pp < &proc[NPROC]; pp++){
      if(pp->parent == p){  // 如果进程 pp 是当前进程的子进程
        // 确保子进程没有处于 exit() 或 swtch() 状态
        acquire(&pp->lock);  // 获取子进程的锁

        havekids = 1;  // 当前进程有子进程
        if(pp->state == ZOMBIE){  // 如果子进程处于僵尸状态
          // 找到一个已退出的子进程
          pid = pp->pid;
          
          // 如果 addr 不为 0，尝试将退出状态（xstate）写入到父进程的内存中
          if(addr != 0 && copyout(p->pagetable, addr, (char *)&pp->xstate,
                                  sizeof(pp->xstate)) < 0) {
            release(&pp->lock);  // 释放子进程的锁
            release(&wait_lock);  // 释放等待锁
            return -1;  // 写入失败，返回 -1
          }

          // 释放子进程资源，包括将 proc 设置为 UNUSED
          freeproc(pp);
          release(&pp->lock);  // 释放子进程的锁
          release(&wait_lock);  // 释放等待锁
          return pid;  // 返回已退出的子进程的 pid
        }
        release(&pp->lock);  // 释放子进程的锁
      }
    }

    // 如果没有子进程，或者当前进程已被杀死，直接返回 -1
    if(!havekids || killed(p)){
      release(&wait_lock);  // 释放等待锁
      return -1;  // 没有子进程，或者进程已被杀死
    }
    
    // 没有找到已退出的子进程，当前进程进入休眠，等待子进程退出
    sleep(p, &wait_lock);  //DOC: wait-sleep
  }
}
```

通过以上的分析，我们可以知道，如果父进程不调用 wait ，而直接退出就会导致资源无法正确释放，而如果子进程不调用 exit ，也会导致资源无法正确释放，但是在比较完善的操作系统中，我们的孤儿进程通常都会被 init 进程接管，而没有显式调用 exit 而退出的进程，也会被操作系统识别为 ZOMBINE 状态而正常释放。

下面再来介绍一下 kill ，这是一个比较特殊的杀死进程，我们仅仅会将它的 killed 表示设置为 1 ，表示它已经被杀死，而不是将他的状态设置为 ZOMBINE 或者 UNUSED ，如果它正处于 SLEEPING 状态，则会唤醒它，使得它可以正常退出。

```c
// 杀死指定 pid 的进程。
// 目标进程不会立即退出，直到它尝试返回用户空间（见 trap.c 中的 usertrap()）。
// 该函数遍历进程表，找到匹配的进程 pid，
// 将其标记为“已杀死”，如果进程处于 SLEEPING（睡眠）状态，
// 将其状态更改为 RUNNABLE（可运行）。进程最终会在返回用户空间时退出。
int
kill(int pid)
{
  struct proc *p;

  // 遍历进程表中的所有进程。
  for(p = proc; p < &proc[NPROC]; p++){
    acquire(&p->lock);  // 获取进程的锁，确保线程安全。
    
    if(p->pid == pid){  // 检查当前进程是否与给定的 pid 匹配。
      p->killed = 1;  // 将进程标记为已杀死（设置标志）。
      
      // 如果进程处于 SLEEPING（睡眠）状态，则将其状态更改为 RUNNABLE（可运行）。
      // 这样可以确保进程被调度运行，并最终退出。
      if(p->state == SLEEPING){
        p->state = RUNNABLE;
      }
      
      release(&p->lock);  // 修改进程状态后释放锁。
      return 0;  // 返回 0 表示操作成功。
    }
    
    release(&p->lock);  // 如果当前进程的 pid 不匹配，释放锁。
  }
  
  return -1;  // 如果没有找到匹配 pid 的进程，返回 -1。
}
```

为什么要延后它的退出？如果立刻将他退出，可能会出现资源上的混乱，比如锁等资源无法被释放等问题，而我们通过这种方式可以正确地，优雅地退出这个进程，而需要做到这一点，需要在合理的位置进行进程是否被杀死的判断。
