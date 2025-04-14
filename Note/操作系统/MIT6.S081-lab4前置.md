# mit6.s081 lab4前置

## 从用户空间陷入

这部分的内容其实在lab3前置的源码阅读部分就已经讨论过了，但是这个是本章的重点，所以再次来讨论一下：

当我们从用户空间执行ecall指令的时候(一般是系统调用的时候会执行)，我们可以在make qemu之后，在user/usys.S中找到这样的代码，以fork为例子：

```assembly
.global fork
fork:
 li a7, SYS_fork
 ecall
 ret
```

他先将我们的需要的系统调用的对象(SYS_fork在这里实际上是int常量，表示这个系统调用的数字)，存入a7，然后通过执行ecall跳转到我们的陷阱处理的地方，由于我们此时是从用户空间陷入，所以我们此时`stvec`寄存器应当是指向用户陷入区的，所以此时是进入到了kernel/trampoline.S的`uservec`部分，其中，前面都是一些保存寄存器，然后获取参数，比较重要的一步就是：

```assembly
csrw satp, t1
```

我们的trampoline在这一步可以将使用的基地址转移到内核页表，但是我们事实上还需要继续执行后续的指令，不可能因为基地址变化，后面的指令就不执行了，这就是操作系统的神奇，用户页中映射的uservec和内核页中映射的uservec的虚拟地址是相同的！随后便会清空TLB，页表缓存，然后跳转到我们的usertrap了！注意，此时我们调用usertrap之后，并不会返回到当前的位置，而是会经过usertrapret来直接返回到用户态。

```c
// 将控制权从内核态恢复到用户态，设置陷阱相关寄存器、页表等必要状态。
// 每次用户进程返回用户态执行时，都会调用这个函数。
void
usertrapret(void)
{
  struct proc *p = myproc();

  // 我们即将把 stvec 设置为 usertrap，也就是用户态 trap 的入口，
  // 所以在返回用户态之前，要先关中断，防止陷阱跳转期间被打断。
  intr_off();

  // 设置 stvec 为 trampoline.S 中的 uservec 函数地址（用户态 trap 的入口地址）。
  // TRAMPOLINE 是 trampoline 映射到高地址的起始地址，uservec 是 tramp.S 中的偏移。
  uint64 trampoline_uservec = TRAMPOLINE + (uservec - trampoline);
  w_stvec(trampoline_uservec);

  // 把接下来用户态再次 trap 到内核时需要用到的内核信息填入 trapframe
  // 这些值会在 trampoline.S 的 uservec 中被用到

  // 保存当前内核页表的 satp 值，在下一次 trap 回内核时要切回这个页表
  p->trapframe->kernel_satp = r_satp();

  // 保存当前进程在内核中的内核栈顶地址，保证正确的栈结构
  p->trapframe->kernel_sp = p->kstack + PGSIZE;

  // 保存 trap 进入内核的入口函数地址（即 usertrap 函数）
  p->trapframe->kernel_trap = (uint64)usertrap;

  // 保存当前 CPU 的 hartid，用于区分不同核心上的进程
  p->trapframe->kernel_hartid = r_tp();


  // 修改 sstatus 寄存器，设置好用户态返回的相关状态
  // 清除 SPP 位（Supervisor Previous Privilege）= 0，表示下一次 sret 返回到 User 模式
  // 设置 SPIE（使能中断）位，表示回到用户态后允许响应中断
  unsigned long x = r_sstatus();
  x &= ~SSTATUS_SPP; // 设置 SPP 为 0，表示 sret 返回后是用户模式
  x |= SSTATUS_SPIE; // 设置 SPIE 为 1，用户态开启中断
  w_sstatus(x);

  // 设置 sepc 寄存器为 trapframe 中保存的用户态 PC（trap 时被保存在 epc 中）
  // sret 执行后就会跳转回这个地址执行
  w_sepc(p->trapframe->epc);

  // 设置即将切换的页表（用户态页表）
  uint64 satp = MAKE_SATP(p->pagetable);

  // 跳转到 trampoline.S 中的 userret 函数，传入用户页表 satp。
  // userret 会完成：
  //   - 切换 satp（用户页表）
  //   - 恢复用户寄存器（包括 sp、ra、a0~a7 等）
  //   - 执行 sret，真正返回到用户态
  uint64 trampoline_userret = TRAMPOLINE + (userret - trampoline);
  ((void (*)(uint64))trampoline_userret)(satp);
}
```

随后，会跳转到userret，它同样处于trampoline之中，用于真正的恢复用户态的信息，然后真正的返回到我们的用户态。



之前在编写系统调用的时候还提到过，我们的artint，artaddr，artfd这几个函数从trapframe中寻找第n个系统调用的参数，并且以对应的格式返回，他们都会调用argraw来检索对应的参数：

```c
static uint64
argraw(int n)
{
  // 获取当前进程
  struct proc *p = myproc();
  
  // 根据参数 n 返回对应的系统调用参数
  switch (n) {
  case 0:
    return p->trapframe->a0;
  case 1:
    return p->trapframe->a1
  case 2:
    return p->trapframe->a2;
  case 3:
    return p->trapframe->a3; 
  case 4:
    return p->trapframe->a4;
  case 5:
    return p->trapframe->a5;
  }
  
  panic("argraw");
  
  return -1;
}
```

我们会发现，这段代码实际上实现地很简单，但是我们还需要注意一点，就是有的时候，我们会传递指针来进行系统调用，内核就需要使用这些指针来读取或写入用户内存，这就会引发问题，而内核实现了安全地将数据传输到用户提供的地址或是安全的从提供的地址处获取数据，比如我们的exec，在传递参数的时候，使用的是传入数组指针的方式，而我们在exec中调用了`fetchaddr`这个函数：
```c
//   addr - 用户空间中待读取的地址。
//   ip - 指向内核空间的指针，读取的 64 位数据将被存储在这里。
int
fetchaddr(uint64 addr, uint64 *ip)
{
  struct proc *p = myproc();

  // 检查地址是否有效。地址必须在进程的虚拟内存范围内，且能够容纳一个 uint64 类型的数据
  if(addr >= p->sz || addr + sizeof(uint64) > p->sz) 
    return -1;
    
  // 从用户空间拷贝数据到内核空间。如果拷贝失败，返回错误
  if(copyin(p->pagetable, (char *)ip, addr, sizeof(*ip)) != 0)
    return -1;
  return 0;
}
```

但是这一部分其实并不算复杂的，我们需要回过去看看我们真正的重点--`copyin`

```c
// 从用户空间拷贝数据到内核空间
//
// 参数：
//   pagetable - 用户进程的页表，用于地址翻译
//   dst       - 目标内核地址，拷贝到这里
//   srcva     - 源地址（用户虚拟地址）
//   len       - 拷贝的字节数
//   本函数会跨页拷贝用户空间的数据。
//   每次调用 walkaddr() 把虚拟地址 srcva 映射为物理地址，
//   然后用 memmove() 拷贝这一页内的内容。
//   如果在任何一步发现地址无效，直接返回 -1。
int
copyin(pagetable_t pagetable, char *dst, uint64 srcva, uint64 len)
{
  uint64 n, va0, pa0;

  while(len > 0){
    // 将 srcva 向下取整到页对齐的地址，得到当前页的起始虚拟地址
    va0 = PGROUNDDOWN(srcva);

    // 使用 walkaddr 将页表中的虚拟地址 va0 映射到物理地址 pa0
    pa0 = walkaddr(pagetable, va0);
    if(pa0 == 0)
      return -1; // 地址无效，说明该页未映射，返回错误

    // 计算从 srcva 开始到当前页末尾还能拷贝多少字节
    n = PGSIZE - (srcva - va0);
    if(n > len)
      n = len; // 如果这一页可拷贝的数据超过需要的长度，截断为 len

    // 从物理地址对应位置拷贝 n 字节数据到内核地址 dst
    // 注意这里 pa0 是页起始物理地址，(srcva - va0) 是页内偏移
    memmove(dst, (void *)(pa0 + (srcva - va0)), n);

    len -= n;

    dst += n;
    srcva = va0 + PGSIZE; // 跨页拷贝，srcva 跳到下一页开头
  }

  return 0;
}
```

可以看见，我们这里出现的熟悉的walk，但是后面跟了个addr，实际上，这里内部依旧是调用的walk，然后计算出其物理地址而已。

随后就是简单的内存数据拷贝了，将计算出来的用户态物理地址中的数据拷贝到目标的内核地址处。

另外还有一个拷贝字符串的操作函数，叫做copyinstr，也是类似的逻辑，没有太大区别。

## 从内核空间陷入

之前我们已经对用户态的陷入做了一些介绍，下面我们来细说一下我们还未涉足的领域--从内核空间陷入。