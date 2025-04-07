# lab-2

## 0. 前置

### **课程记录**

> 操作系统的**隔离性**，举例说明就是，当我们的shell，或者qq挂掉了，我们不希望因为他，去影响其他的进程，所以在不同的**应用程序**之间，需要有隔离性，并且，应用程序和操作系统之间，也是如此。
>
> 当失去操作系统，我们的应用程序将会**直接**与硬件进行交互，数据也将直接存储在**物理内存**中，但是我们的两个应用程序无法识别相互的**边界**，此时就可能会产生**覆盖**，所以，这就是我们希望内存能够隔离的原因，也是我们操作系统所需要实现的功能，另外还需要**multiplexing**(cpu分时复用，也就是多线程，CPU运行一个进程一段时间，再运行另一个进程)
>
> 这里也有一层抽象，操作系统没有直接将cpu提供给应用程序，而是将进程抽象了cpu，这样操作系统才能在多个应用程序之间复用一个或者多个cpu。
>
> 而我们也可以认为exec是对内存的抽象，通过提供exec这样一个系统调用，使得我们可以安全的访问对应的内存，而我们不能够直接访问物理内存。
>
> files，它提供了方便的磁盘抽象，我们唯一与存储系统交互的方式就是files，我们可以任意改写这个文件，而最终，由操作系统来决定，这个files如何与磁盘中的块对应。

---

> 操作系统的**防御性**，操作系统需要确保所有的应用程序都能工作，因此，操作系统不能没有任何抵御攻击的准备，比如说，应用程序向**系统调用**中传递了错误的参数，而导致了操作系统的**崩溃**，进而导致操作系统**拒绝**了为其他所有应用程序提供服务，所以操作系统需要能够应对恶意的程序。
>
> 并且应用程序也不能打破隔离性，而影响到其他的程序，甚至控制内核(很危险)。
>
> 通常来说，**强隔离性**由硬件实现，包括**user/kernel mode**和**虚拟内存**。
---

> 首先，为了支持**user/kernel mode**，处理器也会有两种操作模式，不同的操作模式权限不同，kernel mode可以使用特权命令，而user mode只能使用普通权限命令。
>
> 特权命令，比如说设置page table寄存器，以及处理器的相关状态等。
>
> 当在user mode 执行了特权命令，cpu会拒绝这条命令，并且转入kernel mode，杀死这个进程。
>
> kernel mode还是user mode是通过一个flag寄存器来存储的。
>
> **page table**，每个进程都会有自己的一个独立的page table，也就是说，每个进程都有自己独立的视图，而进程在这个视图上去访问内存空间，从而被映射到物理地址中，这样来保证他们的内存不会重合。

---

> **user/kernel mode**是如何切换的？xv6中，我们在make之后，我们可以在user/usys.S找到ecall这个指令，事实上，通过ecall这个指令，我们能够跳转到内核中指定的由内核控制的位置来执行系统调用。
>
> 举一个例子，当我们调用fork，或者read的时候，事实上没有直接调用函数，而是通过ecall去跳转到内核，执行系统调用。
>
> 现在我们有了可以执行系统调用的手段，之后内核负责实现具体的系统调用，并且需要检查参数，防止被恶意攻击，所以安全可靠无bug的内核也成为TCB(Trusted Computing Base)
>
> **宏内核**：xv6就是一个宏内核的典型代表，所有的操作系统服务都在kernel mode中，由于任何一个操作系统的bug都有可能成为漏洞，而我们有大量的代码放在内核中，出现漏洞的可能性就更高了，这便是宏内核的缺点，而宏内核的另一个代表就是linux，宏内核的优势在于包括文件系统，虚拟内存等子模块都集成在一个程序中，提供了很好的性能。
>
> **微内核**：微内核和宏内核恰恰相反，他将大部分操作系统运行在内核之外，这样可以有效减少内核的代码数量，从而降低出现漏洞的概率，但是微内核也有缺陷，比如说在shell需要与文件系统交互的时候，内核实际上是以消息传递的形式来执行系统调用的，这种情况下，每个操作都需要执行两次跳转，所以性能是更差的。并且，在一个宏内核中，如文件系统和虚拟内存系统可以轻易地共享page cache，但是在微内核中，这些模块都被隔离开了，而这种共享难以实现，也使得难以获得更高的性能。
>
> 到分析xv6启动过程的时候，我发现只看文档，确实还是很不理解的，推荐有实践，带着看代码的部分还是看视频最好。

---

### **xv6手册**

**进程**

进程具有独占的地址空间，以及看上去是仅在运行当前程序的cpu，而其他进程无法对其进行干扰，这样，它就会误以为自己运行在独立一台机器上。

xv6使用页表(硬件实现)为每个进程提供独有的地址空间，而页表将虚拟地址(汇编使用的地址)映射为物理地址(处理器芯片向主存发送的地址)。

从低地址区开始，依次代表着用户(低处存放进程的指令)->全局变量->栈区->堆区

同样的，内核的指令和数据也都被映射到了每个进程的地址空间中的高地址处(为用户留下足够的空间)，当进程使用系统调用的时候，就会跳转到**进程地址空间**的**内核区域**执行(感觉和中断有点像)，而在xv6中，使用的是**proc**来维护的一个进程的状态(包含了页表，内核栈，运行状态等信息)。

每个进程都有自己的用户栈和内核栈，当进程运行用户命令时，只有用户栈被使用，内核栈是空的，在进行系统调用或者中断进入内核的时候，内核代码就会被放入内核栈中执行，用户栈依旧保存着数据，只是现在不活跃，进程的线程交替使用内核栈和用户栈，而用户代码无法操作内核栈，所以即便用户破坏了自己的用户栈，内核也能正常运行，梳理一下流程：

> 进程系统调用 -> 处理器转入内核栈中(或者说指令指针改变到内核栈中) -> 提升硬件特权 -> 运行特权代码 -> 降低特权 -> 转回到用户栈

而我们每个进程都会有一个线程，线程之间的切换，实际上就是挂起当前线程，恢复另一个线程的状态，线程的状态都保存在线程栈上。(有点像中断时的寄存器状态的恢复和保存)

开机后发生的事情：他会初始化自己，将bootloader从磁盘中载入，而bootloader负责将内核从磁盘中载入，随后开始运行`entry`，而entry的最开始，设置了两份页表映射，(低地址的映射和高地址的映射)，因为在最开始，还没有设置页表的时候，我们的机器很可能没有虚拟地址对应的那么大的内存，于是，我们只能将其对应到物理地址上，然后我们会设置这两份页表映射，因为最开始entry还运行在内存的低地址处，所以需要设置 `0:0x400000 -> 0:0x400000`的映射关系，我们还需要设置一个高地址的映射关系KERNBASE:KERNBASE+0x400000 -> 0:0x400000，而kernbase指的就是 0x80000000。

配合这张图会比较好理解。

![figure1-2](C:\Users\chenyue\Desktop\PrOJECT\Notes\Note\操作系统\assets\f1-2.png)

然后entry会继续设置页表，并且将页表的目录`entrypgdir`的物理地址载入`%cr`，此时就可以通过各种操作去实现分页机制了(这里有一些比较底层的没看懂)

最后，entry就需要跳转到内核的C代码，并且在内存的高地址去执行它了

这里我还有很多地方都没看懂，感觉这里还是主要讲的是x86的？有的部分还是自己去看看源代码比较好，我看2024年的源代码，riscv的汇编部分是很少的，也就300行左右，大部分还是c语言，包括很多操作系统的cpu，进程调度啥的，也没看懂，这里分享一下我和AI大战三百回合了解到的东西(不一定对)：

操作系统本质上也是一个进程，也需要一个cpu去执行，而在操作系统启动后，cpu就会不断去执行我们提前写好的代码(执行程序的机器)，而所有cpu的CS:IP，也就是执行程序的指针，都由操作系统这个特殊的进程来调度，但是，如果到最后，没有任何程序可以供cpu执行，那么cpu就会陷入一个空闲循环，因为cpu总是应该执行的，然后，当我们新建了一个进程，需要cpu去执行的时候，就会获取一个cpu，同时修改他的CS:IP使他的指令指针指向这个进程的起始位置，然后将相关的上下文切换，cpu便能继续执行新的工作，当然在后面的调度，我们还会知道，时间片耗尽的中断机制，当然，这是后面的话题了。

其实这方面最开始最困扰我的就是，你的CPU他怎么知道他何时应该执行程序？CPU在操作系统没有启动的时候，不是陷入睡眠吗？启动操作系统之后，就能向下执行命令，cpu难道是个只会工作的机器吗？cpu并不是一开始就在运行吧？我们如果创建一段程序，想要去执行它，除了设置它的状态为RUNNING，并且表示他能够呗调度，但是实际上，cpu怎么知道，他应该去执行什么呢？ 或者说，cpu怎么知道，他何时应该执行命令？ 毕竟即便是汇编代码，想要cpu开始去执行，让他的CS:IP开始偏转，不也需要去运行这个程序吗？然而我们就是把它的状态标记成RUNNING，这合适吗？还有什么其他的步骤？

其实现在一想，自己的一些疑问也有点意思，但是同时理解了之后，回答这些疑问也是比较轻松的，不知道自己和AI大战之后得出来的结论正确没有，如果有什么错误，希望给我能留言!

----

## 1. gdb

1. **谁调用了syscall？** kernel/trap.c中的usertrap()这个函数

2. **p->trapframe->a7的值代表什么意思？** 通过hint，我从initcode.S里面发现，a7寄存器代表了执行系统调用的类型，这里应该也是代表了执行的系统调用类型

3. **cpu之前是什么模式？** 在这里，我输入`p /x $sstatus`打印出的值为`0x200000022`，而当我重新运行，设置断点在usertrap处，输入打印出来的值为`0x200000020`，我调了一下，发现是`intr_on()`这个函数执行之后改变的，感觉应该关系不是很大？

   全英文的那个手册也很劝退，之前的cpu模式应该是用户态吧，目前应该是超级用户(问的AI)

4. **num存在那个寄存器？** 通过对应的sepc，找到对应的panic行，发现num对应了a3这个寄存器，而内核崩溃的原因是因为访问了未映射的地址0，scause的值记录了触发异常的原因。

5. 至于panic的时候，正在运行的进程，我倒是没有找出来，gdb在kernel进入panic的时候，也跟着陷入了，即便是在panic里面打印相对应的进程信息也没有找到答案。

好了，实验完成之后，不要忘记将你的代码恢复原样！

## 2. Trace

这个b实验要求，确实看不懂，我总是想象不出来要如何去修改一些代码，使得能够满足要求，

但是hint还是很有用的，跟着hint来完全可以弄出来。

首先，trace就是说，对于每一个系统调用，你都需要跟踪`调用进程，系统调用的名称，系统调用的返回值`，直接给我说这些东西，我肯定是一头雾水，但是实验确实也给了足够的hint让我们完成这个lab，总体来讲，还是比较简单的。

首先，我们的trace.c貌似已经为我们准备好了？我们在user空间里面只需要修改makefile/usys.pl以及user.h，而trace的函数签名已经在trace.c中确定了：

```c
entry("trace");	//usys.pl

int trace(int); //user.h

$U/_trace\ //makefile
```

接下来，便是在syscall的施工了，最开始，我们需要进入到proc.h修改我们的proc结构体的成员，为其加上`int tracing`的成员，作为tracing的mask掩码。

而后，我们在调用trace系统调用的时候，需要为我们的proc这个tracing参数赋值，所以就在sysproc.c添加我们的trace系统调用是如何执行的：

```c
uint64 sys_trace(void) {
  int mask;
  //从传来的第一个参数中获取mask
  argint(0, &mask);
  myproc()->tracing = mask;
  return 0;
}
```

这里为什么不直接通过参数来获得？ai说这里是xv的特性，就不深究了，argint就是从参数中获取第一个值，并赋给mask，然后为我们的tracing赋值，这样，我们就能够进行判断了!

仅仅是赋值，并不能直接就能够输出我们的进程信息，我们还需要在syscall.c中的syscall函数中，添加合理的输出信息：

```c
void
syscall(void)
{
  int num;
  struct proc *p = myproc();
  num = p->trapframe->a7;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    // Use num to lookup the system call function for num, call it,
    // and store its return value in p->trapframe->a0
    p->trapframe->a0 = syscalls[num]();
      //修改在此处，根据掩码来输出相应的系统调用信息。
    if ((p->tracing >> num) & 1) {
      printf("%d: syscall %s -> %ld\n", p->pid, syscall_str[num], p->trapframe->a0);
    }
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
}
```

当然，我们的每个系统调用都有相对应的数字，所以我们还需要在syscall.h中添加映射：

```c
#define SYS_trace  22
```

并且将我们的映射和对应的系统调用添加进系统调用的数组中：

```c
static uint64 (*syscalls[])(void) = {
[SYS_fork]    sys_fork,
[SYS_exit]    sys_exit,
[SYS_wait]    sys_wait,
[SYS_pipe]    sys_pipe,
[SYS_read]    sys_read,
[SYS_kill]    sys_kill,
[SYS_exec]    sys_exec,
[SYS_fstat]   sys_fstat,
[SYS_chdir]   sys_chdir,
[SYS_dup]     sys_dup,
[SYS_getpid]  sys_getpid,
[SYS_sbrk]    sys_sbrk,
[SYS_sleep]   sys_sleep,
[SYS_uptime]  sys_uptime,
[SYS_open]    sys_open,
[SYS_write]   sys_write,
[SYS_mknod]   sys_mknod,
[SYS_unlink]  sys_unlink,
[SYS_link]    sys_link,
[SYS_mkdir]   sys_mkdir,
[SYS_close]   sys_close,
//添加这一行就可以了
[SYS_trace]   sys_trace,
};

```

然后，我们的syscall_str是未定义的，这里需要我们自己去定义一个数组：

```c
//名称数组索引
static char* syscall_str[] = {
[SYS_fork]    "fork",
[SYS_exit]    "exit",
[SYS_wait]    "wait",
[SYS_pipe]    "pipe", 
[SYS_read]    "read",
[SYS_kill]    "kill",
[SYS_exec]    "exec",
[SYS_fstat]   "fstat",
[SYS_chdir]   "chdir",
[SYS_dup]     "dup",
[SYS_getpid]  "getpid",
[SYS_sbrk]    "sbrk",
[SYS_sleep]   "sleep",
[SYS_uptime]  "uptime",
[SYS_open]    "open",
[SYS_write]   "write",
[SYS_mknod]   "mknod",
[SYS_unlink]  "unlink",
[SYS_link]    "link",
[SYS_mkdir]   "mkdir",
[SYS_close]   "close",
[SYS_trace]   "trace",
};
```

但是通过这样，我们会发现，fork出来的子进程，无法继承父进程的tracing字段，因为我们tracing是新增的，并没有启用任何继承操作，所以在proc.c里面，需要对fork函数进行修改添加以下的内容：

```c
np->tracing = p->tracing; // 继承父进程的跟踪状态
```

这样，我们的trace就完成了！

## 3. attack

这个lab是真的卡了我很久，课上说这次的lab很简单，可是对于我这种菜鸡还是太过困难了。

首先观察attacktest的函数，发现其中生成一个secret，随后调用调用secret.c中的函数，就是将一串密钥写入内存中，后回到attacktest，调用我们需要写的的attack，我们的attack需要将获取的密钥写入到fd2中，以供检查，然后attacktest会比对我们的获取的密钥是否和真正的密钥相同。

总体流程如上，这里需要操作内存页，好难~先来看看我们的secret.c

```c
int
main(int argc, char *argv[])
{
  if(argc != 2){
    printf("Usage: secret the-secret\n");
    exit(1);
  }
  char *end = sbrk(PGSIZE*32);
  end = end + 9 * PGSIZE;
  strcpy(end, "my very very very secret pw is:   ");
  strcpy(end+32, argv[1]);
  exit(0);
}
```

这里分配了32个内存页，随后在第10页中写入了我们的密钥，同时带有一个前缀，我最开始的思路其实就是根据前缀去寻找密钥，但是事实证明，这个方法行不通，于是我开始从其他的突破点入手，比如说，这是一个页，我可以通过找到这个页，并且我们也能够知道其偏移量，这样，我们就有能力去寻找这个密钥，但是如何去寻找到这一页地址？我们如何知道，重新分配的内存页，哪一页才是我们需要的？我想，这就是强迫我们去读源码的一个lab。

去读源码然后分析也是一个很艰难的过程，这里大部分也参考了其他的博客，我一个人肯定是写不出来的😭

首先我们需要了解的是内存页的分配与回收(位于`kernel/kalloc.c`)，xv6的内存是采取栈式的链表管理，分配时，从栈中取出内存页，回收的时候则推回栈顶，这也是我们寻找真正的页的根基。

随后我们需要知道，在attacktest中，attack之前，到底分配了多少，回收了多少内存页，只要知道这个，我们的困难就迎刃而解了。当然，虽然说上去很简单，但这也要求我们必须熟悉其中涉及的每一个系统调用，函数会分配的页数，这也是我们熟悉xv6的一个很好的机会。

首先是attacktest中的fork()，这是一个系统调用，位于`/kernel/proc.c`，总所周知，每一个进程都会分配一个内存页，fork当然也会，当我们进入到fork的函数体的时候，也确实如此，他通过调用allocproc来分配空间，在allocproc中：

```c
  // Allocate a trapframe page.
  if((p->trapframe = (struct trapframe *)kalloc()) == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  // An empty user page table.
  p->pagetable = proc_pagetable(p);
  if(p->pagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }
```

他通过kalloc分配了一个页表，这个页表存储上下文信息，而又通过proc_pagetable分配了一个页，这个页是虚拟页到物理页的映射，在内部，他会创建一个新的页表：

```c
pagetable = uvmcreate();
if (pagetable == 0)
  return 0;
```

然后将虚拟地址空间的一部分映射到物理地址上面：

```c
  // 映射 trampoline 代码（用于系统调用返回）
  // trampoline 是在用户虚拟地址空间的最顶端，负责在系统调用的上下文切换时跳转到内核。
  // 该映射设置为只读且可执行（PTE_R | PTE_X），因为 trampoline 是内核代码
  // 注意：此区域仅用于系统调用跳转，并不是用户程序直接访问的，因此没有 PTE_U 标志
  if(mappages(pagetable, TRAMPOLINE, PGSIZE,
              (uint64)trampoline, PTE_R | PTE_X) < 0){
    uvmfree(pagetable, 0);
    return 0;
  }
  // 映射 trapframe 页，位于 trampoline 页面下方
  // trapframe 用于保存进程的上下文（寄存器值、堆栈指针等），
  // 这是在系统调用或中断时保存进程状态的地方，供 trampoline 使用。
  // 这里的映射设置为可读写（PTE_R | PTE_W），因为操作系统需要在该页写入和读取数据。
  if(mappages(pagetable, TRAPFRAME, PGSIZE,
              (uint64)(p->trapframe), PTE_R | PTE_W) < 0){
    uvmunmap(pagetable, TRAMPOLINE, 1, 0);
    uvmfree(pagetable, 0);
    return 0;
  }
```

我们的xv6是采取的三级页表结构，最开始创建的pagetable作为我们的根页表，我们的trampoline和trapframe(之前已经分配)作为我们的根页表，但是此时还需要一个二级页表来帮助映射，所以我们还需要一个分配一个二级页表来维护完整的映射结构，因此此处分配了三个页表，对应了pagetable，trampoline和二级页表。

回到我们的fork()

```c
  // Copy user memory from parent to child.
  if(uvmcopy(p->pagetable, np->pagetable, p->sz) < 0){
    freeproc(np);
    release(&np->lock);
    return -1;
  }
```

这里将我们分配的内存从父进程复制到子进程中，这里原父进程也占用了四个页表，并且还需要一个一级页表和二级页表来建立映射，所赐此处又新分配了6个页表。

所以我们的fork一共分配了10个页表。

然后回到我们的attacktest，我们可以看见，下一步可能有内存分配的地方就是exec()去执行我们的secret了，exec可以说是最复杂的一个系统调用了，但是我们的目的不是深入分析他，目前只需要知道他分配了哪些内存即可，他在代码前面会使用proc_pagetable去创建新的页表以此来替换旧页表。

```c
  if((pagetable = proc_pagetable(p)) == 0)
    goto bad;
```

随后，exec会遍历elf文件的头表，会将我们的LOAD段加入我们的内存页中，使用`readelf`查看_secret如下：

```bash
root@rinai-VMware-Virtual-Platform:/home/rinai/6S081/xv6-labs-2024# readelf -l user/_secret 

Elf 文件类型为 EXEC (可执行文件)
Entry point 0x5c
There are 3 program headers, starting at offset 64

程序头：
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  RISCV_ATTRIBUT 0x00000000000073e5 0x0000000000000000 0x0000000000000000
                 0x0000000000000047 0x0000000000000000  R      0x1
  LOAD           0x0000000000001000 0x0000000000000000 0x0000000000000000
                 0x0000000000000901 0x0000000000000901  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000001000 0x0000000000001000
                 0x0000000000000000 0x0000000000000020  RW     0x1000

 Section to Segment mapping:
  段节...
   00     .riscv.attributes 
   01     .text .rodata 
   02     .data .bss 
```

结果如上，存在两个LOAD段，需要分别加载到不同的内存页中，(大概知道是咋回事就行)，同时，我们当然还需要一级页表和二级页表来进行映射，因此，这里创建了四个页。

加载段到内存的代码是这样写的：

```c
 // 加载程序段到内存中
  for(i=0, off=elf.phoff; i<elf.phnum; i++, off+=sizeof(ph)){
    if(readi(ip, 0, (uint64)&ph, off, sizeof(ph)) != sizeof(ph))
      goto bad;
    if(ph.type != ELF_PROG_LOAD) // 跳过非 LOAD 类型段
      continue;
    if(ph.memsz < ph.filesz) // 内存大小不能小于文件大小
      goto bad;
    if(ph.vaddr + ph.memsz < ph.vaddr) // 防止溢出
      goto bad;
    if(ph.vaddr % PGSIZE != 0) // 地址必须页对齐
      goto bad;

    // 为段分配内存（虚拟地址）
    uint64 sz1;
    if((sz1 = uvmalloc(pagetable, sz, ph.vaddr + ph.memsz, flags2perm(ph.flags))) == 0)
      goto bad;
    sz = sz1;

    // 从文件中读取代码或数据，写入到段对应虚拟地址中
    if(loadseg(pagetable, ph.vaddr, ip, ph.off, ph.filesz) < 0)
      goto bad;
  }
```

随后还会分配用户栈：

```c
  // 将栈的起始地址 `sz` 向页边界对齐
  sz = PGROUNDUP(sz);
  
  uint64 sz1;
  
  // 为进程分配一块内存区域，作为用户栈空间。
  // uvmalloc 会根据提供的页表 `pagetable` 分配内存，
  // 并将分配的内存区域的结束地址返回给 `sz1`
  // 新的栈空间大小从 `sz` 到 `sz + (USERSTACK + 1) * PGSIZE`，分配的是可写内存 (PTE_W)
  if ((sz1 = uvmalloc(pagetable, sz, sz + (USERSTACK + 1) * PGSIZE, PTE_W)) == 0)
    goto bad;  // 如果分配失败，跳转到错误处理部分

  // 更新栈的结束地址为分配的内存的结束地址
  sz = sz1;
  
  // 清除栈的保护区域，使其变为不可访问
  // 栈的保护区域位于栈的顶部 (栈底的内存页)，
  // 用来防止栈溢出攻击，当访问该区域时会触发页面错误 (page fault)
  uvmclear(pagetable, sz - (USERSTACK + 1) * PGSIZE);
  
  // 设置栈指针 `sp` 为栈的结束地址
  sp = sz;
  
  // 计算栈的基址，即栈的底部地址
  // `stackbase` 指向栈空间的起始位置
  stackbase = sp - USERSTACK * PGSIZE;
```

此时，又分配了两页内存。

最后，还会调用函数来释放旧的页表

```c
void
proc_freepagetable(pagetable_t pagetable, uint64 sz)
{
  uvmunmap(pagetable, TRAMPOLINE, 1, 0);
  uvmunmap(pagetable, TRAPFRAME, 1, 0);
  uvmfree(pagetable, sz);
}
```

其中，trampoline是整个操作系统共享，不会被释放，tramframe也是一样，是用户态和内核态转换时用到的存储区，不会被释放，所以最后会释放旧页表占用的5页和用户的4页内存。

exec一共分配了9页，释放了9页，而后，开始执行我们的secret程序，他申请了32页内存，并且在第10页写入了相对应的secret，到现在，我们的attacktest已经申请过了42页内存。

而secret实际上是作为fork出来的子进程运行的，而在我们的父进程中，首先就是等待secret子进程的退出，此时，我们分配的内存页都会被回收，此处的回收顺序，也应当重点关注(kernel/proc.c)：

```c
// 释放进程使用的资源并重置进程状态
static void
freeproc(struct proc *p)
{
  // 如果该进程的 trapframe（保存用户态寄存器上下文）存在，则释放它占用的内核内存
  if(p->trapframe)
    kfree((void*)p->trapframe);
  // 将 trapframe 指针置为空，表示已释放
  p->trapframe = 0;
  // 如果该进程的页表存在，则释放该页表所管理的所有用户态内存
  if(p->pagetable)
    proc_freepagetable(p->pagetable, p->sz);
  // 清除种种标志
  p->pagetable = 0;
  p->sz = 0;
  p->pid = 0;
  p->parent = 0;
  p->name[0] = 0;
  p->chan = 0;
  p->killed = 0;
  p->xstate = 0;
  p->state = UNUSED;
}
```

可以看到，我们首先释放的是trapfram，随后释放所有的用户态内存：

```c
uvmfree(pagetable_t pagetable, uint64 sz)
{
  if(sz > 0)
    uvmunmap(pagetable, 0, PGROUNDUP(sz)/PGSIZE, 1);
  freewalk(pagetable);
}
```

这个函数会根据地址从低到高释放内存，释放顺序为：data + text段，用户栈+page guard，最后是32页堆内存，此时的空闲页表的顺序为：5页页表，32页页表，用户栈，page guard。

随后再次执行fork，紧接着又是exec系统调用，根据之前的经验，fork会分配10页，而exec分配了9页，释放了9页，数量不变，此时，我通过计算会发现，第17页，就是我们需要寻找的答案：

```c
int main(int argc, char *argv[]) {
  char *end = sbrk(PGSIZE*32);
  end = end + 16 * PGSIZE;
  printf("secret: %s\n", end+32);
  write(2, end+32, 8);
  exit(0);
}
```

这个lab确实感觉很有难度，包括分析的视角，深度，感觉都不是一般人能想到的，锻炼了独立思考的能力(虽然我没有)，还能够理解xv6的源码，感觉是很棒的lab，就是感觉对我这种人来说，太难了💀，开始担心起之后的lab会有多恐怖了。

我一个人肯定写不出这样的lab了，参考文献：

https://nos-ae.github.io/posts/attack-xv6/

https://blog.csdn.net/weixin_42543071/article/details/143351746

