# MIT6.S081-lab3前置

## 前言

在lab2中，我们会为了完成attack这个实验，而花费大量的时间去阅读相关的系统调用源码，以此来分析出我们最终secret所在的页表的位置，而我写lab2中，重点并没有关注其中的逻辑关系，有很多想仔细分析的函数都没有深入地去看，故写此文，以便理解记忆。

## proc

要想搞清楚proc所分配的页表，首先，还得了解我们的proc是什么样子的：

```c
// 每个进程的状态
struct proc {
  struct spinlock lock;       // 保护进程状态的锁

  // 在使用以下字段时必须持有 p->lock 锁：
  enum procstate state;        // 进程的当前状态
  void *chan;                  // 如果非零，表示进程正在 chan 上睡眠
  int killed;                  // 如果非零，表示该进程已经被杀死
  int xstate;                  // 进程退出时返回给父进程的退出状态
  int pid;                     // 进程 ID
  int tracing;                 // 跟踪系统调用的掩码

  // 必须持有 wait_lock 锁才能使用以下字段：
  struct proc *parent;         // 父进程

  // 这些字段是进程私有的，因此不需要持有 p->lock 锁：
  uint64 kstack;               // 内核栈的虚拟地址
  uint64 sz;                   // 进程内存大小（字节数）
  pagetable_t pagetable;       // 用户页表
  struct trapframe *trapframe; // trampoline.S 中使用的数据页面
  struct context context;      // 切换到该进程时的上下文
  struct file *ofile[NOFILE];  // 打开的文件
  struct inode *cwd;           // 当前工作目录
  char name[16];               // 进程名称（用于调试）
};
```

我们的重点便是下半部分，kstack就是我们的内核栈的虚拟地址的指针，当我们进程需要执行系统调用的时候，我们首先会通过trampoline保存我们的上下文数据(寄存器中的数据)，保存到哪？每一个proc都有可能执行系统调用，所以不能将他们存放在同一个地方，所以，我们的proc结构体还维护了一个trapframe字段，用来我保存我们在执行系统调用之前的上下文，以便于返回时恢复状态，我们可以在trampoline.S里面轻易地看见，保存上下文的代码：

```assembly
  ....
  sd ra, 40(a0) 
  sd sp, 48(a0) 
  sd gp, 56(a0)
  sd tp, 64(a0) 
  sd t0, 72(a0) 
  .....
```

其中的a0，表示我们的trapframe的地址，ra/sp等表示我们执行系统调用之前的寄存器，这就表示将这些寄存器保存到我们的trapframe中，他作为一个单独的页表分配，而在返回的时候恢复上下文的代码，我们也可以在trampoline.S中找到：

```assembly
  ...
  ld ra, 40(a0)
  ld sp, 48(a0)
  ld gp, 56(a0)  
  ld tp, 64(a0) 
  ld t0, 72(a0) 
  ...
```

这就表示从trapframe中获取信息，并赋予寄存器。

在这里我们可以重新理一下之前做过的lab2，刚开始，可能很多人都对trapframe和trampoline不是很理解，实际上，trapframe就是我们需要开辟的新空间，为我们进程保存上下文，恢复上下文服务的，而trampoline是我们所有的进程通过用户页表映射到同一个地方，公共的部分，所有的用户页表都会映射到这个位置，而我们的三级页表实际上也需要分配一个页表来将我们的trampoline和trapframe映射到真正的物理地址上面，最终，我们的proc是如下的映射结构：

```c
VPN2[511]
 └── VPN1[511]
      └── VPN0
           ├── [511] → trampoline
           └── [510] → trapframe

```

VPN2是我们根页表，也就是最开始创建的页表，随后调用**mappages**的时候，就会自动创建二级页表，自动创建三级页表(最开始只有VPN2)，然后才能建立我们的映射关系。

所以这里的proc_pagetable一共创建了三个页表，也就可以解释了。

## exec

这也是一个相当有趣且复杂的系统调用，我在做lab的时候，没有看见需要去阅读源码的准备工作，所以最开始就给没看，写起lab来也是很吃力的，在exec是exec之前，首先，他是一个系统调用，我们可以在syscall.c里面找到所有syscall的入口：

```c
void
syscall(void)
{
  int num;
  struct proc *p = myproc();  // 获取当前进程结构体指针
  num = p->trapframe->a7;  // 获取系统调用号（存储在 trapframe 的 a7 寄存器中）
  // 检查系统调用号是否在有效范围内，且是否有对应的系统调用函数
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    // 根据系统调用号 `num` 查找对应的系统调用函数，并调用它
    //这里的syscall[num]就是直接调用了系统调用的函数。
    // 将返回值存储在进程的 trapframe 中的 a0 寄存器中
    p->trapframe->a0 = syscalls[num]();  
    // 如果进程启用了系统调用追踪功能，打印系统调用的详细信息
    if ((p->tracing >> num) & 1) {
      printf("%d: syscall %s -> %ld\n", p->pid, syscall_str[num], p->trapframe->a0);
    }
  } else {
    // 如果系统调用号无效或没有对应的系统调用函数，输出错误信息
    // 并将返回值设置为 -1
    printf("%d %s: unknown sys call %d\n", p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
}
```

我们知道，这里的syscalls数组实际上就是各个系统调用的函数的指针，所以我们通过他，就可以进入到我们的sys_exec了，接下来，就是关于他的代码，而想要去理解这个exec，就得明白，他是干什么的，exec事实上，在lab1里面也见识了很多次

首先是我们执行exec系统调用的入口：

```c
uint64
sys_exec(void)
{
  char path[MAXPATH], *argv[MAXARG];
  int i;
  uint64 uargv, uarg;

  // 获取传入的参数：参数 1 是用户空间的 argv 地址
  argaddr(1, &uargv);
  // 获取程序路径
  if (argstr(0, path, MAXPATH) < 0) {
    return -1;  // 如果路径获取失败，返回 -1
  }
  memset(argv, 0, sizeof(argv));  // 初始化 argv 数组

  // 从用户空间获取 argv 数组
  for (i = 0;; i++) {
    if (i >= NELEM(argv)) {
      goto bad;
    }
    // 获取用户空间中每个参数的地址
    if (fetchaddr(uargv + sizeof(uint64) * i, (uint64*)&uarg) < 0) {
      goto bad; 
    }
    // 如果参数为 0，表示 argv 数组结束
    if (uarg == 0) {
      argv[i] = 0;
      break;
    }
    // 为 argv[i] 分配内存
    argv[i] = kalloc();
    if (argv[i] == 0)
      goto bad;
    // 获取用户空间中字符串并复制到内核空间
    if (fetchstr(uarg, argv[i], PGSIZE) < 0)
      goto bad;
  }
  // 此处是真正的执行exec函数，加载指定的程序
  //下面我们会详细介绍
  int ret = exec(path, argv);
  // 释放 argv 数组中分配的内存
  for (i = 0; i < NELEM(argv) && argv[i] != 0; i++)
    kfree(argv[i]);
  return ret;  // 返回 exec 的返回值，成功则为 0，失败则为 -1
bad:
  // 如果有任何错误发生，释放所有已分配的内存
  for (i = 0; i < NELEM(argv) && argv[i] != 0; i++)
    kfree(argv[i]);
  return -1;  // 返回 -1 表示执行失败
}
```

我们可以看见，我们系统调用的入口做了一系列的准备工作，然后执行了exec之后，从内核态返回，这里返回就不是执行原来的程序了，而是执行我们在exec中加载的程序！当然，前提是exec执行成功了。

随后，我们可以来看看我们的exec到底干了什么，首先，我们的exec在最开始初始化了一大堆值，我们目前先不关心他，我们要知道，在最开始，我们的exec调用了

```c
  begin_op();
```

这个函数，有啥用？事实上，每一个文件操作之前，前面都会跟上一个begin_up，下面是他的具体实现：

```c
void begin_op(void)
{
  // 获取日志锁，确保对日志操作的独占访问
  acquire(&log.lock);
  // 循环直到可以开始新的文件系统操作
  while(1) {
    // 如果当前日志正在提交，等待提交完成
    if (log.committing) {
      // 当前日志正在提交，休眠并等待提交完成
      sleep(&log, &log.lock);
    } 
    // 如果当前操作可能会超出日志的空间，等待日志提交
    else if (log.lh.n + (log.outstanding + 1) * MAXOPBLOCKS > LOGSIZE) {
      // 当前操作可能耗尽日志空间，休眠并等待提交
      sleep(&log, &log.lock);
    } 
    else {
      // 当前操作可以进行，增加待处理操作的计数
      log.outstanding += 1;
      // 释放日志锁，允许其他线程访问日志
      release(&log.lock);
      // 成功开始文件系统操作，跳出循环
      break;
    }
  }
}
```

我们可以看到，事实上，这就是一个加上一个锁的操作，而是通过日志加锁，来保证每个文件操对日志操作的原子性，顺序性，但是这里竟然是全局锁，哈人。通过加这样一个锁，我们对日志的操作就实现了原子性，随后，我们会从文件系统中寻找我们需要执行的系统调用：

```c
  if((ip = namei(path)) == 0){
    end_op();
    return -1;
  }
  ilock(ip);
```

ilock实际上就是锁定该文件，防止其他进程修改

随后，我们需要读取这个文件，然后对这个文件进行一些验证，随后分配空间，在这里需要注意，trapframe依旧是旧的，之后会进行更改：

```c
  // 从 inode 所代表的文件中读取 ELF 头部（ELF Header）到 elf 结构体中
  // 参数含义：
  //   ip：表示打开的文件 inode
  //   0：表示这是从用户空间发起的（传 0）
  //   (uint64)&elf：读取到内存 elf 的地址
  //   0：从文件开头开始读
  //   sizeof(elf)：读取字节数等于 ELF 头的大小
  if(readi(ip, 0, (uint64)&elf, 0, sizeof(elf)) != sizeof(elf))
    goto bad;  // 如果读取失败或读取长度不足，则跳转到错误处理
  // 检查读取到的 ELF 文件头的魔数是否匹配
  // ELF_MAGIC 是一个特定的常数（0x464C457F），用来判断文件是否为 ELF 格式
  if(elf.magic != ELF_MAGIC)
    goto bad;  // 如果不是 ELF 文件，即魔数不对，跳转到错误处理
  // 为当前进程 p 创建一个新的用户页表
  // 旧的页表包含的是之前进程的地址空间，现在要执行新程序，因此需要一个全新的页表
  // 这里会映射trampoline和trapframe，但是trapframe在之后会进行更新。
  if((pagetable = proc_pagetable(p)) == 0)
    goto bad;  // 分配失败时跳转到错误处理
```

之后，还会进行一系列验证，最终，会将我们的elf文件的段数据写入内存，总体验证过程和流程如下：

```c
for(i=0, off=elf.phoff; i<elf.phnum; i++, off+=sizeof(ph)){
    // 从 ELF 文件中读取每个程序头（Program Header）到 ph 结构体
    if(readi(ip, 0, (uint64)&ph, off, sizeof(ph)) != sizeof(ph))
        goto bad;
    // 如果程序头的类型不是 ELF_PROG_LOAD（表示可加载的程序段），跳过当前程序头
    if(ph.type != ELF_PROG_LOAD)
        continue;
    // 程序段的内存大小必须大于或等于文件大小，否则不合法，跳转到错误处理
    if(ph.memsz < ph.filesz)
        goto bad;
    // 如果程序段的结束地址小于起始地址，说明程序头出错，跳转到错误处理
    if(ph.vaddr + ph.memsz < ph.vaddr)
        goto bad;
    // 程序段的起始地址必须是页面大小对齐的，否则跳转到错误处理
    if(ph.vaddr % PGSIZE != 0)
        goto bad;
    // 为程序段分配内存，确保分配的内存区域是连续的，并且不会与其他程序段冲突
    uint64 sz1;
    if((sz1 = uvmalloc(pagetable, sz, ph.vaddr + ph.memsz, flags2perm(ph.flags))) == 0)
        goto bad;
    sz = sz1;
    // 将 ELF 程序段的数据加载到内存中
    if(loadseg(pagetable, ph.vaddr, ip, ph.off, ph.filesz) < 0)
        goto bad;
}
```

一旦发现错误，就会立刻跳转到错误处理的地方，而最后，我们会将loadseg加载到页表中：

```c
static int
loadseg(pagetable_t pagetable, uint64 va, struct inode *ip, uint offset, uint sz)
{
  uint i, n;
  uint64 pa;
  // 遍历程序段的每一页（PGSIZE大致为4KB）
  for(i = 0; i < sz; i += PGSIZE){
    // 获取当前虚拟地址（va + i）对应的物理地址
    pa = walkaddr(pagetable, va + i);
    // 如果无法找到对应的物理地址，发生错误
    if(pa == 0)
      panic("loadseg: address should exist");
    // 判断当前页是否是最后一页，如果是，则计算实际读取的大小
    if(sz - i < PGSIZE)
      n = sz - i;
    else
      n = PGSIZE;
    // 从文件（inode）中读取数据到物理地址 pa 上
    if(readi(ip, 0, (uint64)pa, offset+i, n) != n)
      return -1;  // 如果读取失败，返回错误
  }
  return 0;  // 成功加载所有数据
}
```

实际上，loadseg对于我们并不是过于重要，我们只需要知道，他会将我们给的段写入到单独的内存页，这样就行了。

那么，将段单独写入内存页的逻辑在哪？其实我们已经见识过了：

```c
for(i=0, off=elf.phoff; i<elf.phnum; i++, off+=sizeof(ph))
```

这里的for循环，就是将我们的程序段循环遍历的地方，通过这样，我们可以将符合要求的段加入我们的页表中，即便我们不知道有多少个段，我们也可以加一个printf语句来帮助我们判断，从而弄清楚到底分配了几个页。

至此，我们就搞清楚了我们的页表是如何分配的了

但是，到目前为止，我们仅仅是将我们的页表给分配了，并且将我们的段都写入进去了，我们还需要分配我们的栈空间。

```c
  // 将当前地址 sz 向上对齐到页边界，为用户栈分配内存页（至少 USERSTACK + 1 页）
  // 第一页作为 "栈保护页"，不可访问；剩下的 USERSTACK 页是实际的用户栈
  sz = PGROUNDUP(sz);  // 保证下一次页分配是对齐的
  uint64 sz1;
  if((sz1 = uvmalloc(pagetable, sz, sz + (USERSTACK+1)*PGSIZE, PTE_W)) == 0)
    goto bad; // 分配失败，跳转错误处理
  sz = sz1;

  // 清除栈底的 guard 页，即使映射存在也让它不可访问（trap）
  uvmclear(pagetable, sz-(USERSTACK+1)*PGSIZE);

  // 设置 sp（栈指针）和 stackbase（栈底，栈增长不会低于这里）
  sp = sz; // 初始 sp 指向用户栈顶部
  stackbase = sp - USERSTACK*PGSIZE; // 栈的最低地址（不包括 guard 页）
```

当然，我们可以看见除了我们的栈空间，还分配了栈保护页，这是为了防止栈溢出是将其他区域的数据覆盖掉所设置的，当发生栈溢出，访问到保护页的时候，系统就会触发异常。

随后需要初始化我们的栈

```c
// 将命令行参数字符串逐个压入用户栈，同时构建一个 ustack[] 数组，
// 存放每个参数字符串在用户栈中的地址（即 argv[i] 的指针）。
for(argc = 0; argv[argc]; argc++) {
  // 超出限度，就转到错误处理
  if(argc >= MAXARG)
    goto bad;
  // 给这个参数字符串分配空间，长度需要加上结尾的 '\0'
  sp -= strlen(argv[argc]) + 1;
  // 内存对齐
  sp -= sp % 16;
  // 如果当前栈指针已经低于栈底，说明栈空间不够，返回错误
  if(sp < stackbase)
    goto bad;
  // 把参数字符串从内核拷贝到用户页表的栈空间（虚拟地址为 sp）
  if(copyout(pagetable, sp, argv[argc], strlen(argv[argc]) + 1) < 0)
    goto bad;
  // 将我们每个参数的指针放在数组里面。
  ustack[argc] = sp;
}
// 添加 NULL 结尾，相当于 argv[argc] = 0，表示参数结束
ustack[argc] = 0;
```

可以看见，我们主要将我们的参数通过sp拷贝到用户的栈空间中，并且初始化了一个ustack数组来存储这些sp指针，他在之后会显现他的作用：

```c
// 这里是为我们需要拷贝的数据腾出栈空间
// 所以需要改变指针栈指针
  sp -= (argc+1) * sizeof(uint64);
// 内存对齐
  sp -= sp % 16;
  if(sp < stackbase)
    goto bad;
// 将我们的ustack数组的指针数组拷贝到栈空间中，我们之后便能通过ustack中存储的sp指针
// 去找到对应的参数
  if(copyout(pagetable, sp, (char *)ustack, (argc+1)*sizeof(uint64)) < 0)
    goto bad;
```

综上，我们将ustack拷贝到栈空间，随后我们就可以轻易地通过这个数组存储的指针，访问到相对应的参数了，为啥要通过ustack访问？这是为了保证我们栈访问的安全性，统一性。

最后，便是我们的收尾工作了，我们的exec真的要结束了：

```c
// 将栈指针 (sp) 保存到 trapframe 的 a1 寄存器。
// 这个栈指针将在用户程序中使用，'sp' 指向用户栈。
p->trapframe->a1 = sp;

// 保存程序名以便调试。
// 从完整路径中提取程序名（即最后一个 '/' 后的部分）。
for(last = s = path; *s; s++)
    if (*s == '/')          // 查找路径中的最后一个 '/'
        last = s + 1;       // 程序名从 '/' 后开始
safestrcpy(p->name, last, sizeof(p->name));  // 将程序名复制到进程名称中，方便调试

// 提交新的用户空间镜像，通过切换页表和更新进程状态来完成。
// 保存当前的（旧的）页表
oldpagetable = p->pagetable;
// 设置新的页表
p->pagetable = pagetable;
// 更新进程的大小
p->sz = sz;
// 设置进程的初始程序计数器（epc）为 ELF 文件的入口点
p->trapframe->epc = elf.entry;  // 初始程序计数器 = main 函数的地址
// 设置栈指针（sp）为用户空间的栈指针
p->trapframe->sp = sp; // 初始栈指针
// 释放旧的页表资源
proc_freepagetable(oldpagetable, oldsz);
```

我们这里主要是将栈指针的起始位置存入我们的trapframe，a0保存的是我们的系统调用号，而a1则是我们的参数列表的栈指针。这一步已经完成了，最后的，那就是我们的首位工作，将各个参数更新，除此之外，还需要将我们的旧的页表给释放掉，值得注意的是，我们这里替换的页表实际上是用户态的页表，而当前的代码是在内核态中执行的，所以我们的exec不会受到用户态的替换页表的影响。

## 系统调用到底干了啥？

另外讲点有意思的，我们之前提过无数次用户态和内核态，我们的用户态通过执行系统调用，就会跳转到内核态，那么到底是怎么跳转的？我们现在就来解决这个问题：

在我们执行系统调用的时候，实际上，在用户态代码中，我们的这些调用都会在user.h中声明，在编译之后，usys.S会通过usys.pl这个脚本去生成出来，这样，就实现了我们在用户态执行系统调用的跳转的汇编代码，举个例子，比如说exec：

```assembly
exec:
 li a7, SYS_exec
 ecall
 ret
.global open
```

我们将SYS_exec存入a7这个寄存器中，SYS_exec是啥？事实上，在我们的lab2中应该已经能够知道，SYS_exec就是在kernel/syscall.h中定义的一个常量，而这里也确实引入了这样一个头文件，所以，这里是直接存入了我们的系统调用编号，方便调用的！

然后调用ecall，会跳转到我们的内核指定的位置，也就是stvec寄存器指向的位置，在哪里指定的？当然是在初始化我们的xv6的时候指定的，我们可以在trap.c中找到通过调用函数来设置stvec的函数w_stvec，其内部是这样的：

```c
w_stvec(uint64 x)
{
  asm volatile("csrw stvec, %0" : : "r" (x));
}
```

通过这个函数，我们可以动态的修改我们的stvec，因为我们的ecall不仅有可能在用户态执行，还可能在内核态执行，所以通过这种方式，当我们从usertrap中返回的时候，我们就会把这个cpu的stvec寄存器设置成用户产生中断的处理位置，这样，就可以让我们的ecall跳转到正确的位置。

此时我们是在用户态中调用的ecall，所以应该是跳转到trampoline.S中的uservec代码段中，如下：

```assembly
.globl uservec
uservec:    
        # 保存用户空间传入的 a0 寄存器值到 sscratch 寄存器，
        # 因为 a0 会用于获取 TRAPFRAME 地址。
        csrw sscratch, a0
        # 设置 a0 为 TRAPFRAME 的地址，
        # 每个进程都有一个单独的 trapframe 内存区域，
        # 但是在每个进程的用户页表中，该区域被映射到相同的虚拟地址。
        li a0, TRAPFRAME
     
        ...
        #此处省略很多行保存寄存器逻辑
        # 将之前保存的用户空间的 a0 值（即 TRAPFRAME 的地址）重新保存到 p->trapframe->a0。
        csrr t0, sscratch
        sd t0, 112(a0)
        # 从 p->trapframe->kernel_sp 加载内核栈指针，并设置 sp。
        ld sp, 8(a0)
        # 从 p->trapframe->kernel_hartid 加载当前的 hartid，设置 tp。
        ld tp, 32(a0)
        # 从 p->trapframe->kernel_trap 加载 usertrap 函数的地址，并跳转到 usertrap。
        ld t0, 16(a0)
        # 从 p->trapframe->kernel_satp 获取内核页表地址，设置 satp 寄存器。
        ld t1, 0(a0)
        # 等待先前的内存操作完成，以便后续的操作使用用户页表。
        sfence.vma zero, zero
        # 安装内核页表（切换到内核页表）。
        csrw satp, t1
        # 刷新 TLB 缓存，移除过时的用户页表条目。
        sfence.vma zero, zero
        # 跳转到 usertrap 函数，该函数不会返回。
        jr t0
```

可以看见，通过一系列相关操作，我们就会跳转到usertrap(C语言函数)中，而usertrap就是进入到syscall之前需要做的准备工作，但是此时已经真正的进入内核态了！

 下面来看看我们的usertrap究竟是什么个事

```c
// 处理来自用户态的中断、异常或系统调用。
// 被 trampoline.S 中的 uservec 调用。
void
usertrap(void)
{
  int which_dev = 0;
  // 检查是否真的是从用户态陷入内核的。
  // 如果 SPP（Supervisor Previous Privilege）不为 0，说明不是从用户态过来的，出错。
  if((r_sstatus() & SSTATUS_SPP) != 0)
    panic("usertrap: not from user mode");
  // 设置 stvec 为内核态的中断向量 kernelvec，表示从现在开始内核态中断交由 kernelvec 处理
  // 这是为了防止在内核执行期间再次陷入时，还用的是用户态的 uservec。
  w_stvec((uint64)kernelvec);
  // 获取当前运行的进程
  struct proc *p = myproc();
  // 保存当前触发 trap 时的 PC（sepc 存储的是触发 trap 的用户态地址）
  // 这里实际上也可以在trampoline里面就执行保存操作
  // 而其他的用户寄存器则必须要在汇编trampoline里面保存
  p->trapframe->epc = r_sepc();
  // 判断是不是系统调用（scause=8 表示 ecall-from-U-mode）
  if(r_scause() == 8){
    // 如果进程已经被标记为 killed，则直接退出
    if(killed(p))
      exit(-1);
    // ecall 指令执行后 sepc 仍然指向 ecall，所以这里 +4 是为了从下一条指令继续执行
    p->trapframe->epc += 4;
    // 开中断（因为 syscall 处理期间我们已经不需要再屏蔽中断了）
    intr_on();
    // 执行系统调用
    syscall();
  } else if((which_dev = devintr()) != 0){
    // 如果是设备中断（如时钟中断、UART 等），交给 devintr() 处理
    // 返回值标识具体设备，这里先保存下来以便后续判断是否需要 yield CPU
    // 如果是设备中断，正常返回；否则下面打印错误信息
  } else {
    // 既不是 syscall 也不是设备中断，属于非法中断或异常
    printf("usertrap(): unexpected scause 0x%lx pid=%d\n", r_scause(), p->pid);
    printf("            sepc=0x%lx stval=0x%lx\n", r_sepc(), r_stval());
    // 标记进程为 killed，稍后会终止它
    setkilled(p);
  }
  // 如果此时进程被标记为 killed，就退出
  if(killed(p))
    exit(-1);
  // 如果是时钟中断，就让出 CPU（进行调度）
  if(which_dev == 2)
    yield();
  // 从内核返回用户态（设置好寄存器、trapframe、切换 stvec）
  usertrapret();
}
```

每一步相关的说明都已经注释好了，在usertrap中，我们真的会调用一个syscall函数，并且在调用完成之后，真的会调用usertrapret返回到我们的用户态，当然，如果我们执行的是exec系统调用，返回的用户态已经是被替换过的全新的程序，至此，我们已经实现了一个系统调用的闭环。

目前来看，我们已经理解了很多之前不知道的东西，现在，我们来阅读阅读页表课前置要求的一些源码吧！

## sbrk/kalloc/mappages

其实最好的阅读方式并不是通过硬啃一个文件，一段源代码能够解决的，关键是流程，我们应该跟着流程来分析，首先是我们在lab2中用到的sbrk，我们可以在syscall.c中找到数组中的sbrk，我们可以通过它跳转到sys_sbrk的函数：

```c
uint64
sys_sbrk(void)
{
  uint64 addr;
  int n;

  argint(0, &n);
  addr = myproc()->sz;
  if(growproc(n) < 0)
    return -1;
  return addr;
}
```

这里的argint事实上也是获取参数，然后将我们传入的参数向下传入给growproc函数，而这个函数是干啥的？我们可以仔细来看看这个函数：

```c
int
growproc(int n)
{
  uint64 sz;
  struct proc *p = myproc();
  // 获取当前进程的虚拟内存大小（sz表示进程的堆顶）
  sz = p->sz;
  if(n > 0){
    // 如果 n > 0，表示需要增加内存
    // 调用 uvmalloc 分配从 sz 到 sz + n 的内存
    if((sz = uvmalloc(p->pagetable, sz, sz + n, PTE_W)) == 0) {
      return -1;  // 分配失败，返回 -1
    }
  } else if(n < 0){
    // 如果 n < 0，表示需要减少内存
    sz = uvmdealloc(p->pagetable, sz, sz + n);  // 释放 sz 到 sz + n 之间的内存
  }
  // 更新进程的虚拟内存大小（sz）
  p->sz = sz;
  return 0;  // 返回 0 表示成功
}
```

在这里，我们可以看见至关重要的一个函数，uvmalloc，之前做lab2分析的时候，无论在哪里都可以看见他的身影，虽然我通过注释标出了他的功能，但是我们应该继续往下看，他到底是干了个什么事！

```c
uint64
uvmalloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz, int xperm)
{
  char *mem;
  uint64 a;
  int sz;
  // 如果新的内存大小小于当前大小，则返回当前大小
  if(newsz < oldsz)
    return oldsz;
  // 将 oldsz 对齐到页边界（上舍入到最接近的页边界）
  oldsz = PGROUNDUP(oldsz);
  // 从 oldsz 到 newsz 分配内存，每次分配一个页
  for(a = oldsz; a < newsz; a += sz){
    sz = PGSIZE;  // 每次分配一个页面
    mem = kalloc();  // 分配一个物理页
    if(mem == 0){
      // 如果分配失败，释放已经分配的内存并返回 0
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
#ifndef LAB_SYSCALL
    // 如果没有启用 LAB_SYSCALL，清零分配的内存
    memset(mem, 0, sz);
#endif
    // 将我们当前分配的内存页到进程的虚拟地址空间
    if(mappages(pagetable, a, sz, (uint64)mem, PTE_R|PTE_U|xperm) != 0){
      // 如果映射失败，释放内存并退回
      kfree(mem);
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
  }
  // 返回新的内存大小
  return newsz;
}
```

很明显，这里uvmalloc主要干了两件事：

1. 分配内存
2. 映射虚拟地址

我们现在有的是时间，便可以好好的看一下kalloc是干了一个什么事，mappages又是在干啥：

```c
// kalloc: 从内核的空闲内存链表中分配一个页面（PGSIZE）。
// 它通过加锁来确保线程安全，并从空闲链表中取出一个空闲的内存块（大小为页面）。
// 如果成功，它将返回该内存块的地址。如果没有空闲内存块，则返回 NULL。
void *kalloc(void)
{
  struct run *r;
  // 获取锁，保证访问内存分配器时的线程安全。
  acquire(&kmem.lock);
  // 从空闲内存链表中获取一个空闲页面块
  r = kmem.freelist;
  // 如果有空闲内存块，更新空闲链表指针
  if(r) {
    kmem.freelist = r->next;
  }
  // 释放锁
  release(&kmem.lock);
  // 如果没有定义 LAB_SYSCALL，则将分配的内存填充为 5（用垃圾数据填充）
#ifndef LAB_SYSCALL
  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
#endif
  // 返回分配的内存块地址，类型转换为 void*。
  return (void*)r;
}

```

在这里，大多数信息已经通过注释标记好了，在lab2中我们提过，我们xv6的内存是一种栈的形式保存的，事实上，只是一个比喻，而真正的数据结构是链表，我们的kalloc便是同空闲内存链表中获取一块页，这样，而这个去除的规则恰好是符合栈的形式的，值得注意的是，我们的kalloc分配的是物理地址，因此我们需要通过mappages来映射这个物理地址！

那么我们已经知道，我们的页表是如何分配的了，下面就可以看看我们的mappages的逻辑了：

```c
// mappages 函数将从虚拟地址 va 开始的内存区域映射到物理地址 pa 开始的物理内存区域。
// 每个页表项（PTE）对应一个虚拟地址页，perm 参数指定了映射的权限。
// va 和 size 必须是页对齐的，表示映射的虚拟地址和大小都应该是页面大小的整数倍。
// 返回 0 表示成功，返回 -1 表示无法为虚拟地址分配页表页面。

int mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
  uint64 a, last;   // a 是当前处理的虚拟地址，last 是最后一个虚拟地址
  pte_t *pte;        // 页表项指针
  // 确保虚拟地址 va 和大小 size 都是页对齐的
  if((va % PGSIZE) != 0)
    panic("mappages: va not aligned");  // 如果虚拟地址没有页对齐，触发 panic
  if((size % PGSIZE) != 0)
    panic("mappages: size not aligned");  // 如果大小没有页对齐，触发 panic
  // 如果大小为 0，表示没有映射的内容，触发 panic
  if(size == 0)
    panic("mappages: size");
  a = va;              // 当前虚拟地址从 va 开始
  last = va + size - PGSIZE;  // 计算最后一个需要映射的虚拟地址，确保是页对齐的
  for(;;) {
    // 使用 walk 函数获取虚拟地址 a 所对应的页表项（pte），如果页表项没有找到，则分配新的页表项
    if((pte = walk(pagetable, a, 1)) == 0)
      return -1;  // 如果无法为当前虚拟地址分配页表项，则返回 -1
    // 检查该页表项是否已经被映射（即 PTE_V 标志位是否已设置）
    if(*pte & PTE_V)
      panic("mappages: remap");  // 如果该页表项已存在映射，触发 panic
    // 将该页表项的值设置为物理地址 pa 的页表项，并设置权限（perm）和有效位（PTE_V）
    *pte = PA2PTE(pa) | perm | PTE_V;
    // 如果当前虚拟地址是最后一个虚拟地址，则退出循环
    if(a == last)
      break;
    a += PGSIZE;  // 移动到下一个虚拟地址（下一个页面）
    pa += PGSIZE; // 移动到下一个物理地址（下一个页面）
  }
  return 0;  // 返回 0 表示映射成功!
}
```

基本没有什么特别重要的，就是检查各种条件，其中的重点便是walk这个函数，通过walk，去获取我们的虚拟地址，并且在漫步过程中，他会自己创建不存在的页表项(三级页表结构)，这个walk，便是我们映射页表的核心逻辑了，下面是对walk的详细注释：

```c
// 返回页表 pagetable 中对应虚拟地址 va 的 PTE（页表项）的地址。
// 如果 alloc != 0，则在需要时创建缺失的页表页。
//
// RISC-V 的 Sv39 分页方案使用了三级页表。
// 每个页表页包含 512 个 64 位的页表项（PTE）。
// 一个 64 位的虚拟地址被划分为五个字段：
//   位 39..63 —— 必须为零（当前 Sv39 只使用 0~38 位，共 39 位虚拟地址）。
//   位 30..38 —— 9 位，用作第 2 级页表的索引。
//   位 21..29 —— 9 位，用作第 1 级页表的索引。
//   位 12..20 —— 9 位，用作第 0 级页表的索引。
//   位  0..11 —— 12 位，用作页内偏移（即页面内的字节偏移量）。
pte_t *walk(pagetable_t pagetable, uint64 va, int alloc)
{
  // 如果虚拟地址 va 超出了最大允许的虚拟地址范围，则触发 panic
  if(va >= MAXVA)
    panic("walk");
  // 遍历页表层级，通常有 3 层页表（根页表、二级页表和三级页表）
  for(int level = 2; level > 0; level--) {
    // 根据虚拟地址 va 和当前页表层级 level 计算出当前页表项的索引
    pte_t *pte = &pagetable[PX(level, va)];
    // 如果该页表项有效（即 PTE_V 标志位为 1），则继续向下遍历
    if(*pte & PTE_V) {
      // 获取下一级页表的物理地址，并更新 pagetable 为该页表
      pagetable = (pagetable_t)PTE2PA(*pte);
        //这部分可以不用管
//#ifdef LAB_PGTBL
//     // 如果当前页表项是叶子节点（即最后一层页表），则返回该页表项
//      if(PTE_LEAF(*pte)) {
//        return pte;
//      }
#endif
    } else {
      // 如果页表项无效，且 alloc 为真，则分配一个新的页表
      // 如果分配失败，则返回 NULL
      if(!alloc || (pagetable = (pde_t*)kalloc()) == 0)
        return 0;
      // 初始化新分配的页表为零
      memset(pagetable, 0, PGSIZE);
      // 将当前页表项的值设置为新分配页表的物理地址，并设置有效位 PTE_V
      *pte = PA2PTE(pagetable) | PTE_V;
    }
  }
  // 返回当前页表项的指针，该页表项对应的是虚拟地址 va 的最后一级页表项
  return &pagetable[PX(0, va)];
}
```

~~啊啊啊xv6你是一个香香软软的小蛋糕~~，我们可以看见，我们之前提过很多次的三级页表，终于在这里露出了鸡脚，这里的三个for循环，分别代表每一级的页表，**`PX(2, va)`**：提取虚拟地址 `va` 中的 `VPN2` 部分的索引，随后通过我们的 `PTE2PA` 讲对应的页表项转换成下一级页表的物理地址，从而获取对应的页表，进而进行下一次转换， **`PX(1, va)`**，提取虚拟地址 `va` 中的 `VPN1` 部分的索引，**`PX(0, va)`**，提取虚拟地址 `va` 中的 `VPN0` 部分的索引，也就是我们最后一级页表所对应的虚拟地址的页表项。

这里，每一级页表，都会通过kalloc分配一个物理页，但是实际上，最终只有叶子页表上的页表才会映射到我们的物理地址中，其实到这里，还需要再引用一下xv6手册的知识。

> `kvmmap`(***kernel/vm.c\***:127)调用`mappages`(***kernel/vm.c\***:138)，`mappages`将范围虚拟地址到同等范围物理地址的映射装载到一个页表中。它以页面大小为间隔，为范围内的每个虚拟地址单独执行此操作。对于要映射的每个虚拟地址，`mappages`调用`walk`来查找该地址的PTE地址。然后，它初始化PTE以保存相关的物理页号、所需权限（`PTE_W`、`PTE_X`和/或`PTE_R`）以及用于标记PTE有效的`PTE_V`(***kernel/vm.c\***:153)。
>
> 在查找PTE中的虚拟地址（参见图3.2）时，`walk`(***kernel/vm.c\***:72)模仿RISC-V分页硬件。`walk`一次从3级页表中获取9个比特位。它使用上一级的9位虚拟地址来查找下一级页表或最终页面的PTE (***kernel/vm.c\***:78)。如果PTE无效，则所需的页面还没有分配；如果设置了`alloc`参数，`walk`就会分配一个新的页表页面，并将其物理地址放在PTE中。它返回树中最低一级的PTE地址(***kernel/vm.c\***:88)。
>
> 上面的代码依赖于直接映射到内核虚拟地址空间中的物理内存。例如，当`walk`降低页表的级别时，它从PTE (***kernel/vm.c\***:80)中提取下一级页表的（物理）地址，然后使用该地址作为虚拟地址来获取下一级的PTE (***kernel/vm.c\***:78)。
>
> `main`调用`kvminithart` (***kernel/vm.c\***:53)来安装内核页表。它将根页表页的物理地址写入寄存器`satp`。之后，CPU将使用内核页表转换地址。由于内核使用标识映射，下一条指令的当前虚拟地址将映射到正确的物理内存地址。

以上为xv6手册中对我们的Walk及其原理相关的介绍，简单来说，walk应该如何去寻找这些页表的索引，其实并不是由我们规定的，而是由硬件规定的这种查询办法，由于我们的cpu在使用虚拟地址的时候，就是采取这种办法去找到对应的物理地址，所以我们的walk也需要采取相同的办法，为什么硬件已经实现了查找功能，但是我们操作系统还需要去实现walk？因为我们最初在操作系统上分配页表，映射页表，是需要操作系统去将他映射到对应的位置的，我觉得这部分是比较难理解的，所以说最好还是去看原文的手册，当然，2020年的课程也有对应的学生提这个问题，可以看看教授是怎么说的。

我们可以看看PX到底是什么

```c
#define PGSHIFT 12
#define PXMASK          0x1FF // 9 bits
#define PXSHIFT(level)  (PGSHIFT+(9*(level)))
#define PX(level, va) ((((uint64) (va)) >> PXSHIFT(level)) & PXMASK)
```

这里的主要作用，其实就是从虚拟地址va中提取索引值，首先是将其右移`层级 * 9 + 12`然后与mask相与计算得到9位数字，对应我们之前说过的PTE为9位，返回的就是我们的索引值，然后从这个索引值中找到对应的表项。

并且，在当前找到的表项有效的时候，还会将这个表项的地址左移十位(原因见手册，这里是为了去除标志位)，然后右移12位(同在手册中可见，12位刚好是我们的页表的尺寸，这样就能够腾出offset，表示真正的页表)：

```c
    if(*pte & PTE_V) {
      pagetable = (pagetable_t)PTE2PA(*pte);
```

最终，循环结束，我们的最终会得到一个第三级的页表，然后我们在mappages中调用了

```c
    *pte = PA2PTE(pa) | perm | PTE_V;
```

这是至关重要的一步，我们可以注意到，我们之前的pte基本都是赋值的形式，这是因为，之前都是为了找到我们的地址，而现在真的找到了我们需要的虚拟地址，然后我们就可以解引用，将我们需要映射物理地址给他映射到这个三级页表中，也就是加入这个三级页表的表项！！！至此，我们的映射就完成了。

这里总结一下感受到的好处，像二级页表，三级页表实际上是通过kalloc分配的物理内存，所以会占用我们的内存，，但是比起`页偏移量| offset`的形式，还是能够通过懒加载节省不少空间，并且寻址页变得更灵活，感觉分页还是很神奇的。

**cpu咋根据虚拟地址找到你的物理地址？**

对于CPU，我们的汇编中的所有代码涉及到的地址都仅仅是虚拟地址，而不是物理地址，然而，我们如何通过这样一个虚拟地址去找到我们的物理地址呢？在cpu中，有一个寄存器叫做satp，用来保存一个地址(实际上就是最高层级的地址，也就是进程的页表地址)，作为我们找到最终物理地址的参考，通过这个寄存器存储的地址，我们的cpu可以告诉MMU，从哪里可以找到将虚拟内存地址翻译成物理地址的"表单"，事实上，MMU就是充当了一个翻译器，它计算地址的功能实际上和walk函数是一样的，这样就能够实现准确的查找，而mmu属于硬件部分，不是我们分析的对象。

到了这里，我们的vm.c已经分析了一大半了。

在进行下一步之前，在这里记录一些概念：

## **TLB页表缓存**

我们在之前涉及的函数中可以看到，我们的cpu是从最高级的页表进行查找对应的物理页，其中会涉及到三次跳转，索引都是存在物理页中的，所以需要读取三次内存，代价很高，所以实际上，我们的cpu都会对最近使用过的虚拟地址的翻译结果有缓存，这个缓存就好做**页表缓存TLB**，而TLB对于我们操作系统是不可见的，不是我们讨论的范畴，这里仅作了解，当然，如果cpu切换了我们的页表，我们过去缓存的的TLB当然也就不可用了，需要被清空，在riscv中，清空TLB的指令为：`sfence_vma`，我们可以在xv6中为数不多的几个汇编源文件中找到这个指令，比如`trampoline.S`。

插曲就到这里，下面我们涉足的地方将是关于内核的页表！

## kvminit/kvminithart

事实上，内核的页表在启动的时候就会分配，我们可以在通过entry，一步一步找到我们的这两个函数的调用：

> **entry**->**start.c**->**main.c**

首先，由于在启动xv6的时候，我们并没有映射物理地址，所以说会通过

```c
  w_satp(0);
```

来禁用分页，在之后，我们就会通过物理地址直接返回到我们的main函数了。

我们可以在main.c中找到这样的代码：

```c
    kvminit();       // create kernel page table
    kvminithart();   // turn on paging
```

首先，就来看看我们的kvminit()

```c
void
kvminit(void)
{
  kernel_pagetable = kvmmake();
}
```

可以看到，这里主要是调用了kvmmake函数来分配内核页表，进一步下去：

```c
// 构建一个 kernel 的页表（direct-map 页表，虚拟地址 = 物理地址）。
// 内核代码、数据、外设、trampoline、内核栈都在此页表中映射。
pagetable_t
kvmmake(void)
{
  pagetable_t kpgtbl;
  // 通过kalloc分配页来作为内核的根页表
  kpgtbl = (pagetable_t) kalloc();
  memset(kpgtbl, 0, PGSIZE);  // 清零页表
  // 映射 UART 外设的寄存器地址（物理地址 = 虚拟地址）
  // 用于串口输出，例如 printf 打印信息
  kvmmap(kpgtbl, UART0, UART0, PGSIZE, PTE_R | PTE_W);
  // 映射 virtio 磁盘接口（内核需要访问硬盘控制器）
  kvmmap(kpgtbl, VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);
//以下不是我们目前需要关心的
//#ifdef LAB_NET
//
//  kvmmap(kpgtbl, 0x30000000L, 0x30000000L, 0x10000000, PTE_R | PTE_W);
//
//  kvmmap(kpgtbl, 0x40000000L, 0x40000000L, 0x20000, PTE_R | PTE_W);
#endif  
  // 映射 PLIC（中断控制器），允许内核收发中断
  kvmmap(kpgtbl, PLIC, PLIC, 0x4000000, PTE_R | PTE_W);
  // 映射内核代码段：可读、可执行（不可写，防止内核被修改）
  kvmmap(kpgtbl, KERNBASE, KERNBASE, (uint64)etext - KERNBASE, PTE_R | PTE_X);
  // 映射内核数据段：从 etext 到 PHYSTOP 的物理内存，可读写
  // 包括 bss 段、内核堆栈、内核堆等
  kvmmap(kpgtbl, (uint64)etext, (uint64)etext, PHYSTOP - (uint64)etext, PTE_R | PTE_W);
  // 映射 trampoline 页（trap 跳板）到最高虚拟地址处
  // trap handler 通过它进入内核；这是统一入口/出口
  kvmmap(kpgtbl, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);
  // 为每个进程分配一个内核栈，并在页表中映射（stack for kernel mode）
  proc_mapstacks(kpgtbl);
  // 返回构造好的页表
  return kpgtbl;
}
```

可以看见，我们的kvmmake实际上就是新分配一个物理页表，随后进行一系列映射，将我们的页表都映射到内核的数据段，代码段所在的物理地址，使得我们可以进行访问。

我们可以进一步看看kvmmap是怎么做的：

```c
void
kvmmap(pagetable_t kpgtbl, uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kpgtbl, va, sz, pa, perm) != 0)
    panic("kvmmap");
}
```

很明显，就是调用了我们之前说过的mappages，然后还会通过walk去找到三级页表，并且将物理页表的地址加到对应的表项中。

我们还看见了一个函数叫做proc_mapstacks,实际上就是为我们每个进程分配内核栈

```c
void proc_mapstacks(pagetable_t kpgtbl)
{
  struct proc *p;
  // 遍历所有进程（proc 是进程表的数组，NPROC 是进程的最大数量）
  for(p = proc; p < &proc[NPROC]; p++) {
    // 分配一页物理内存给每个进程的内核栈
    char *pa = kalloc(); // kalloc 是一个内存分配函数，分配一页物理内存
    if(pa == 0)
      panic("kalloc"); // 如果分配失败，则触发 panic，表示内存不足
    // 计算该进程的内核栈的虚拟地址，KSTACK 是一个宏，通常用于计算栈的位置
    uint64 va = KSTACK((int) (p - proc)); // KSTACK 将进程的索引转化为虚拟地址
    // 将分配到的物理内存页映射到该进程的内核栈的虚拟地址
    kvmmap(kpgtbl, va, (uint64)pa, PGSIZE, PTE_R | PTE_W);
    // kvmmap 是将物理地址映射到虚拟地址的函数，PTE_R | PTE_W 表示该页具有读写权限
  }
}
```

这里在一开始，就为每个可能会被创建的进程分配了进程的内核栈，但是我看2020年的没有这样做。

随后，我们便可以回归到我们的kvminithart了！

```c
void
kvminithart()
{
  // 清除所有旧的 TLB 项。
  // 以确保任何对页表的修改对 CPU 可见。
  // 这里的sfence_vma函数，实际上就是调用我们的riscv的sfence_vma命令！
  sfence_vma();
  // 将satp的值设置为我们刚刚分配的kernel页表
  w_satp(MAKE_SATP(kernel_pagetable));
  // 再次清除 TLB 中的条目，以防设置 satp 后仍存在旧的映射。
  sfence_vma();
}
```

到这里，我们实际上就是启用了我们的分页模式了。

简单总结一下，我们的xv6启动过程中，在分页上面干了什么事？

> 1. 链接的时候，start，main都是被识别为符号，此时还没有开启分页，编译器将他们识别为物理地址，而我们可以直接通过这些符号进行跳转，跳转到我们的main中。
> 2. 在我们的main中，会调用kvminit和kvminithart来初始化我们的内核页表。
> 3. kvminit先分配了一个主页表，随后将内核中的各种信息存放的物理地址映射到这个新建的页表中，并且提前为所有可能分配的进程创建内核栈。
> 4. kvminithart将启用我们的分页设置。

事实上，在我们修改satp的之后，我们的世界就改变了，但是在内核中，依旧可以正常工作，因为kernel page的映射关系中，虚拟地址到物理地址是完全相等的，这里比较特殊，我们的MMU依旧能够将其映射到相同的地址上，这里并不是特殊处理了，依旧只是通过 MMU 去找到的对应的地址，实际上，只是因为我们的内核页表映射的位置设计得很巧妙，如果你还是不理解怎么回事，你可以回去看看kvminit的映射地址时传入的参数。

---

