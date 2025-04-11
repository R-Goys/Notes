# lab3

注：本篇lab的前置知识在《xv6部分源码阅读-1》

## 1. Inspect a user-process page table

实验要求是解释我们通过输入`pgtbltest`输出的包含的内容及其权限位，我的输出如下：

```c
print_pgtbl starting
va 0x0 pte 0x21FC885B pa 0x87F22000 perm 0x5B
va 0x1000 pte 0x21FC7C17 pa 0x87F1F000 perm 0x17
va 0x2000 pte 0x21FC7807 pa 0x87F1E000 perm 0x7
va 0x3000 pte 0x21FC74D7 pa 0x87F1D000 perm 0xD7
va 0x4000 pte 0x0 pa 0x0 perm 0x0
va 0x5000 pte 0x0 pa 0x0 perm 0x0
va 0x6000 pte 0x0 pa 0x0 perm 0x0
va 0x7000 pte 0x0 pa 0x0 perm 0x0
va 0x8000 pte 0x0 pa 0x0 perm 0x0
va 0x9000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFF6000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFF7000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFF8000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFF9000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFFA000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFFB000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFFC000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFFD000 pte 0x0 pa 0x0 perm 0x0
va 0xFFFFE000 pte 0x21FD08C7 pa 0x87F42000 perm 0xC7
va 0xFFFFF000 pte 0x2000184B pa 0x80006000 perm 0x4B
print_pgtbl: OK
```

后面的输出有问题，但是实验要求是解释用户页表，刚好用户页表的信息也能正常输出就不需要管了。

pgtbltest的main函数主体如下：

```c
int
main(int argc, char *argv[])
{
  print_pgtbl();
    //下面执行不了
  ugetpid_test();
  print_kpgtbl();
  superpg_test();
  printf("pgtbltest: all tests succeeded\n");
  exit(0);
}
```

我们这里只有一个函数是我们需要关心的，那么我们可以来看看print_pgtbl干了些什么：

```c
void
print_pgtbl()
{
  printf("print_pgtbl starting\n");
    // 打印前10页的页表条目
  for (uint64 i = 0; i < 10; i++) {
    print_pte(i * PGSIZE);
  }
  uint64 top = MAXVA/PGSIZE;
    // 打印后10页的页表条目
  for (uint64 i = top-10; i < top; i++) {
    print_pte(i * PGSIZE);
  }
  printf("print_pgtbl: OK\n");
}
```

我们可以看到，这里分别打印了前十页页表条目和后十页页表条目，然后我们跟踪到print_pte

```c
void
print_pte(uint64 va)
{	//通过系统调用，获取我们的条目
    pte_t pte = (pte_t) pgpte((void *) va);
    printf("va 0x%lx pte 0x%lx pa 0x%lx perm 0x%lx\n", va, pte, PTE2PA(pte), PTE_FLAGS(pte));
}
```



为了更深刻的理解这个pgpte，我们可以追踪系统调用来弄清楚输出的东西到底是啥，此处，我找到了系统调用pgpte的入口，让我们看看他干了些什么：

```c
int
sys_pgpte(void)
{
  uint64 va;
  struct proc *p;  
	//获取当前进程
  p = myproc();
   // 获取参数，就是当前的虚拟地址偏移
  argaddr(0, &va);
    //通过当前进程的用户页表取执行系统调用。
  pte_t *pte = pgpte(p->pagetable, va);
  if(pte != 0) {
      return (uint64) *pte;
  }
  return 0;
}
```

随后的内容，让我大失所望，结果就是个walk

```c
pte_t*
pgpte(pagetable_t pagetable, uint64 va) {
  return walk(pagetable, va, 0);
}
```

walk函数的具体功能，我已经在上一篇文章中讲过，那么我们直接开始解释吧！

事实上，这些输出实在是一大堆，16进制也不好看，所以我们来重点分析第一个！

> ```c
> va 0x0 pte 0x21FC885B pa 0x87F22000 perm 0x5B
> ```
>
> **虚拟地址 (`va 0x0`)**：
>
> - `va 0x0` 表示虚拟地址是 `0x0`。
>
> **页表项 (`pte 0x21FC885B`)**：
>
> - `pte 0x21FC885B` 是该虚拟地址对应的页表项值。页表项包含了虚拟地址到物理地址的映射关系，以及相关的权限信息，我们还可以将其分为两部分：
>
>   - **物理页框地址（Physical Page Frame Address）**：`pte[63:10]`
>
>   - **权限位和控制位**：`pte[9:0]`
>
> 对于 `0x21FC885B`，我们可以按位来分析：
>
> `0x21FC885B` -> `0010 0001 1111 1100 1000 1000 0101 1011`（二进制表示）
>
> - **物理页框地址**（假设是 `pte[63:10]`）：`0x87F22000`（即将高 22 位作为物理页框地址）
>
> - **权限位和控制位**（假设是 `pte[9:0]`）：
>
>   - **有效位**（`PTE_V`）：`1`，表示该页表项有效。
>
>   - **读写权限**（`PTE_R`）：`1`，表示该页是可读的，通过下一位我们可以发现是不可写的。
>
>   - **用户访问权限**（`PTE_X`）：`1`，表示可以执行。
>
>   - **执行权限**（`PTE_U`）：`1`，表示该用户态可以访问。
>
>     其他倒不是特别重要。
>
> **物理地址 (`pa 0x87F22000`)**：
>
> - `pa 0x87F22000` 是映射后的物理地址。这表示虚拟地址 `0x0` 被映射到了物理地址 `0x87F22000`。也就是说，当程序访问 `0x0` 时，实际访问的是物理地址 `0x87F22000`。
>
> **权限位 (`perm 0x5B`)**：
>
> - `perm 0x5B` 是页表项的权限位，表示该页的访问权限。
>
> `0x5B` -> `0101 1011`（二进制表示）
>
> 根据上文，我们可以直接得出：
>
> - **有效位**（`PTE_V`）：`1`，表示该页表项有效。
> - **读写权限**（`PTE_R`）：`1`，表示该页是可读的，通过下一位我们可以发现是不可写的。
> - **用户访问权限**（`PTE_X`）：`1`，表示可以执行。
> - **执行权限**（`PTE_U`）：`1`，表示该用户态可以访问。
>
> 结合位掩码 `0x5B`，表示该页是有效的、可读的，但可执行，并且用户态可以访问。

---

## 2. Speed up system calls

简单来说，就是为添加一个用户可读的页表，来存储我们的pid，使得可以快速访问，首先，我们先来观察我们的实际的函数是如何调用的：

```c
// 测试用户态系统调用 ugetpid() 是否返回与 getpid() 一致的进程 ID
void
ugetpid_test()
{
  int i;
  printf("ugetpid_test starting\n");
  testname = "ugetpid_test";
  // 多个数据测试
  for (i = 0; i < 64; i++) {
    int ret = fork();  // 创建一个子进程

    if (ret != 0) {  // 父进程执行的逻辑
      wait(&ret);    // 等待子进程结束，并获取退出状态
      if (ret != 0)  // 如果子进程非正常退出，测试失败
        exit(1);
      continue;      // 继续循环，创建下一个子进程
    }
    // 检查内核态 getpid() 与用户态 ugetpid() 返回的 PID 是否一致
    if (getpid() != ugetpid())
      err("missmatched PID");
    exit(0);
  }
  printf("ugetpid_test: OK\n");
}
```

而进入到我们的`ugetpid`可以发现，他就是在我们的用户页的对应的索引位置找到的pid的信息，以此来实现加速，甚至我们的pid页所在的位置，已经给我们规定好了。

```c
int
ugetpid(void)
{
  struct usyscall *u = (struct usyscall *)USYSCALL;
  return u->pid;
}
```

我们进入到USYSCALL定义的地方：

```c
#define USYSCALL (TRAPFRAME - PGSIZE)

struct usyscall {
  int pid;  // Process ID
};
```

显然，我们的usyscall结构体仅仅为我们准备了pid，并且USYSCALL的在页表中映射的位置已经规定好了，剩下就是我们去实现了！

**下面开始！**

通过hint可以发现，我们可以通过在proc结构体字段里面加上我们需要的结构体`usyscall`，也就是我们缓存的结构体，在proc.h中的结构体中加上如下字段：

```c
  struct usyscall *syscall;    // lab3 getpid
```

然后，当我们创建进程的时候，我们肯定需要初始化这个结构体，一般来说，为进程分配页表时，会为其添加各种页表映射(包括trampoline，trapframe)，所以作为附带品，我们的结构体也应当分配一个页，然后映射到内存中，由于在用户页的偏移已经规定好了，所以这并不是问题，所以在allocproc中：

```c
  // 实验部分
  if ((p->syscall = (struct usyscall*)kalloc()) == 0) {
    freeproc(p);
    release(&p->lock);
    return 0;
  }
  // 写入数值
  p->syscall->pid = p->pid;
```

显然，虽然我们可以直接创建一个页，再将结构体写入页中，但是这样显然比较麻烦，所以直接将页分配给结构体即可。

到了这里，确实已经分配好页了，但是还没有映射，在我们的proc_pagetable中还需要建立映射关系：

```c
  if(mappages(pagetable, USYSCALL, PGSIZE,
              (uint64)(p->syscall), PTE_R | PTE_U) < 0) {
    uvmunmap(pagetable, TRAPFRAME, 1, 0);
    uvmfree(pagetable, 0);
    return 0;
  }
```

这里使用的字段，都是之前规定好的，所以无需再去考虑，然后我们还需要考虑这个新分配的页的销毁！这里真的是个很麻烦的过程，找来找去真的累人。

proc_freepagetable()

```c
void
proc_freepagetable(pagetable_t pagetable, uint64 sz)
{
  uvmunmap(pagetable, TRAMPOLINE, 1, 0);
  uvmunmap(pagetable, TRAPFRAME, 1, 0);
  // 取消映射并回收页表
  uvmunmap(pagetable, USYSCALL, 1, 0);
  uvmfree(pagetable, sz);
}
```

freeproc()

```c
static void
freeproc(struct proc *p)
{
  // 释放我们的syscall这一页表
  if(p->syscall)
    kfree((void*)p->syscall);
  p->syscall = 0;
  if(p->trapframe)
    kfree((void*)p->trapframe);
  p->trapframe = 0;
  if(p->pagetable)
    proc_freepagetable(p->pagetable, p->sz);
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

这样，我们的speed up完成了！

## 3. Print a page table

实验要求，简而言之就是需要我们使用kpgtbl系统调用的时候，需要我们去完善其中的一个函数`vmprint`，需要我们以下面的格式打印页表信息：

```c
page table 0x0000000087f22000
 ..0x0000000000000000: pte 0x0000000021fc7801 pa 0x0000000087f1e000
 .. ..0x0000000000000000: pte 0x0000000021fc7401 pa 0x0000000087f1d000
 .. .. ..0x0000000000000000: pte 0x0000000021fc7c5b pa 0x0000000087f1f000
 .. .. ..0x0000000000001000: pte 0x0000000021fc70d7 pa 0x0000000087f1c000
 .. .. ..0x0000000000002000: pte 0x0000000021fc6c07 pa 0x0000000087f1b000
 .. .. ..0x0000000000003000: pte 0x0000000021fc68d7 pa 0x0000000087f1a000
 ..0xffffffffc0000000: pte 0x0000000021fc8401 pa 0x0000000087f21000
 .. ..0xffffffffffe00000: pte 0x0000000021fc8001 pa 0x0000000087f20000
 .. .. ..0xffffffffffffd000: pte 0x0000000021fd4c13 pa 0x0000000087f53000
 .. .. ..0xffffffffffffe000: pte 0x0000000021fd00c7 pa 0x0000000087f40000
 .. .. ..0xfffffffffffff000: pte 0x000000002000184b pa 0x0000000080006000
```

啥意思啊？简而言之，就是将每一级的页表，给他按照树的形式打印出来，详细的要求为：第一行显示 的 `vmprint` 参数。之后，每个 PTE 都有一行，包括引用树中更深处的页表页面的 PTE。每个 PTE 行都缩进了一个数字， `" .."` 表示其 depth 在树中。 每个 PTE 行显示其虚拟地址、pte 位和 从 PTE 中提取的物理地址。 不要打印无效的 PTE。 在上面的示例中， 顶级页表页包含条目 0 和 255 的映射。 下一个 条目 0 的 level down 仅映射了索引 0，而 bottom-level 对于该索引 0 映射了一些条目。

下面开始实验，首先，我们找到对应的函数vmprint，根据hint，我们需要从freewalk寻找相关的线索，freewalk：

```c
// 递归释放页表中所有的非叶子页表项所指向的页表，最终释放自身。
// 这个函数不能用于释放包含实际映射（即叶子节点）的页表。
void
freewalk(pagetable_t pagetable)
{
  // 页表有 2^9 = 512 个页表项（PTE）
  for(int i = 0; i < 512; i++){
    pte_t pte = pagetable[i];
    // 如果该页表项有效 (PTE_V) 且不是叶子节点（没有 R/W/X 权限）
    if((pte & PTE_V) && (pte & (PTE_R | PTE_W | PTE_X)) == 0){
      // 当前页表项指向的是下一级页表
      uint64 child = PTE2PA(pte);             // 获取子页表的物理地址
      freewalk((pagetable_t)child);           // 递归释放子页表
      pagetable[i] = 0;                        // 清空当前页表项
    } 
    // 如果是有效的叶子节点（即具有 R/W/X 权限）
    else if(pte & PTE_V){
      // 不允许释放包含实际映射的叶子节点
      panic("freewalk: leaf");
    }
  }
  // 释放当前这一页页表
  kfree((void*)pagetable);
}
```

众所周知，我们就是需要去寻找每一个PTE，并且将持有这些PTE的页表的虚拟地址打印出来，并且输出其PTE，以及物理地址，而这里有用的信息是什么？我们可以看见，它通过遍历每一个页表项，去查询我们的每一个页表，然后检验其是否有效，是否为叶子节点，如果是叶子节点，那么就可以将其清空，如果不是，递归到下一层。

很显然，我们需要借鉴他的递归思路，遍历思路，以及检查叶子节点的方法，于是：

我们的vmprint还需要另外构造一个函数，总体如下：

```c
void
vmprint(pagetable_t pagetable) {
  printf("page table %p\n", pagetable);
  PRINT(pagetable, 2, 0);
}
```

我们额外需要构造的PRINT函数：

```c
// pagetable: 当前层级的页表指针
// level: 当前层级（2是顶层，0是页表最低层）
// va: 当前页表项对应的虚拟地址起始值
void PRINT(pagetable_t pagetable, int level, uint64 va) {
  uint64 sz;
  // 根据当前层级计算该层级每个页表项的虚拟地址的跨度
  if(level == 2) {
    sz = 512 * 512 * PGSIZE;
  } else if(level == 1) {
    sz = 512 * PGSIZE; 
  } else {
    sz = PGSIZE; 
  }
  // 遍历该页表的512个PTE
  for(int i = 0; i < 512; i++, va += sz) {
    pte_t pte = pagetable[i];
    // 无效，不管他
    if((pte & PTE_V) == 0)
      continue;
	// 树形结构(假)
    for(int j = level; j < 3; j++) {
      printf(" ..");
    }
    // 打印当前页表项的信息：
    // - 虚拟地址 va（从 pagetable 起开始累加）
    // - 页表项内容（pte）
    // - 对应的物理地址（通过 PTE2PA 提取）
    printf("%p: pte %p pa %p\n", 
           (pagetable_t)va, 
           (pagetable_t)pte, 
           (pagetable_t)PTE2PA(pte));
    // 如果该页表项不是叶子节点（即没有 R/W/X 权限），说明它指向下一级页表
    // 递归进入下一级页表打印，否则就不管他，直接走就行。
    if((pte & (PTE_R | PTE_W | PTE_X)) == 0) {
      PRINT((pagetable_t)PTE2PA(pte), level - 1, va);
    }
  }
}
```

## 4. Use superpages

这个实验真的比较难下手，虽然看提示还是能够写出来一点，但是有的东西我真的难写出来😶‍🌫️，这个实验在后面的很多都是参考了别人的博客的。

首先，superpage，其实就是在需求的内存比较大的时候，直接分配一个大页，也不是很多个页，而我们正要去实现这个大页分配的功能。

随后，我们可以查看hint中提醒我们去看看super_test：

```c
// 测试超级页是否能够正确分配、读写，并在子进程中继承
void
superpg_test()
{
  int pid;
  printf("superpg_test starting\n");
  testname = "superpg_test"; // 设置当前测试的名字，用于错误报告中识别
  // 尝试向用户空间申请 N 字节内存（N = 2MB），返回的是当前堆的结束位置
  char *end = sbrk(N);
  if (end == 0 || end == (char*)0xffffffffffffffff)
    err("sbrk failed");  // 分配失败则报错并退出
  // 将堆的结束地址对齐到 2MB 边界（用于保证是超级页起始地址）
  uint64 s = SUPERPGROUNDUP((uint64) end);
  // 检查这个地址 s 是否是有效的超级页（映射正确、可读写、PTE 一致等）
  supercheck(s);
  // 创建子进程，验证页表继承是否正确
  if((pid = fork()) < 0) {
    err("fork"); // 创建失败
  } else if(pid == 0) {
    // 子进程中再次检查超级页是否仍然有效
    supercheck(s);
    exit(0);
  } else {
    int status;
    wait(&status); // 等待子进程退出
    if (status != 0) {
      exit(0); // 子进程出错则退出
    }
  }
  // 如果父子进程都验证成功，则测试通过
  printf("superpg_test: OK\n");  
}
```

显然，这里主要是通过sbrk分配了2MB的大页，如果单纯去使用PAGESIZE，势必会分配很多页，但是只需要使用一个超级页，便可以分配完成，然后我们可以来看看supercheck干了些什么：

```c
// 检查以 s 为起点的超级页（2MB）内所有页的 PTE 是否一致、有效、具有读写权限；并测试读写是否正确
void supercheck(uint64 s) {
  pte_t last_pte = 0;  // 用于记录上一页的 PTE 值（用于比较是否一致）
  // 遍历整个超级页（2MB = 512 个 4KB 页 = 512 * PGSIZE）
  for (uint64 p = s; p < s + 512 * PGSIZE; p += PGSIZE) {
    // 获取虚拟地址 p 对应的页表项
    pte_t pte = (pte_t) pgpte((void *) p);
    // 页表项为空，说明没有映射，报错
    if (pte == 0)
      err("no pte");
    // 如果不是第一次检查，比较当前页表项是否和前一页一致
    if ((uint64) last_pte != 0 && pte != last_pte) {
      err("pte different");
    }
    // 检查页表项是否设置了有效位、读权限、写权限
    if ((pte & PTE_V) == 0 || (pte & PTE_R) == 0 || (pte & PTE_W) == 0) {
      err("pte wrong");
    }
    // 更新 last_pte
    last_pte = pte;
  }
  // 向超级页的每个页写入一个整数值（每页写一个 4 字节的整数 i）
  for (int i = 0; i < 512; i += PGSIZE) {
    *(int*)(s + i) = i;
  }
  // 读取验证：检查写入的值是否被正确保存在内存中
  for (int i = 0; i < 512; i += PGSIZE) {
    if (*(int*)(s + i) != i)
      err("wrong value");
  }
}
```

通过以上函数，我们可以获得的信息：

> 1. 超级页的尺寸为512 * PGSIZE。
> 2. 我们的检测实际上就是去分配一个超级页，看看是否能够正常使用这个超级页。
> 3. 持有超级页的进程fork子进程的时候，必须分配超级页。
> 4. 能够合理分配和释放一个超级页。

以上，就是本次实验的要求。

开始！

根据题目，我们可以知道，要从sbrk入手，我们先来回忆一下sbrk到底干了些啥

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

是不是大概能够会想起之前的源码的介绍了？我们通过growproc来为进程增加内存空间：

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

此处就是我们需要做一定修改的地方，需要为他添加一定的逻辑，使得当申请的内存超过一定大小可以分配超级页，超级页作为一页，肯定不能继续沿用之前所用的普通的page链表，所以在此处做修改之前，先要把我们的准备工作给做好，就是添加一个全局的空闲的超级页链表，但是在此之前，我们还需要去看一个东西，就是普通空闲链表是如何分配的，来看看：

```c
void
kinit()
{
  // 加锁，保证链表插入正常
  initlock(&kmem.lock, "kmem");
  // 记住这里的两个参数
  freerange(end, (void*)PHYSTOP);
}

void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  // 对齐
  p = (char*)PGROUNDUP((uint64)pa_start);
  // 初始化所有的空闲内存为链表
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}
```

我们可以看到，我们的空闲链表的内存来自于[end, PHYSTOP]这段空间，end就是内核已经使用的空间的结尾，而PHTSTOP就是内存的上限，在memlayout中定义。

既然如此，我们的超级页也应当在这段空间里面分配，并且我们还需要一定程度上对kinit进行修改，使得可以添加超级页的空闲链表。

在memlayout中：

```c
// 这里是程序的起始位置
#define KERNBASE 0x80000000L
// 分配的最大的空间，这里规定了最大内存范围
#define PHYSTOP (KERNBASE + 128*1024*1024)
// 这里最多提供12个超级页，虽然实际上是无法提供这么多的，因为内核代码占了一部分。
// 防止分配太多把普通页表的空间占完了
#define SUPERBASE (KERNBASE + 512 * PGSIZE * 12)
```

我们添加了一个SUPERBASE字段，表示在普通页的空间中分配12个超级页。

随后，我们根据规定的范围去修改我们的kinit相关的函数：

```c
struct {
  struct spinlock lock;
  struct run *freelist;
} kmem, supermem;
// 为链表新增超级页实例

void
kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)SUPERBASE);
  superinit();
  // 这里的end是内核结束的地方，之后直到SUPERBASE都是普通页表可以分配的空间
  // 而SUPERBASE到PHYSTOP是超级页表可以分配的空间
}

void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}

// 新增的init函数，初始化超级页的空闲链表
void superinit(){
  initlock(&supermem.lock, "supermem");
  char *p;
  p = (char*)SUPERPGROUNDUP((uint64)SUPERBASE);
  for(; p + SUPERPGSIZE <= (char*)PHYSTOP; p += SUPERPGSIZE)
    superfree(p);
}

// 超级页的释放，直接根据kfree照葫芦画瓢就行了。
void superfree(void *pa) {
  struct run *r;
  // 改成超级页的尺寸，定义在SUPERPGSIZE， 这里爆红是正常的
  if(((uint64)pa % SUPERPGSIZE) != 0 || (char*)pa < SUPERBASE || (uint64)pa >= PHYSTOP)
    panic("superfree");
  // memset，将内存为都置1，表示处于空闲中
  memset(pa, 1, SUPERPGSIZE);
  
  r = (struct run*)pa;
  // 加锁，保证链表插入顺序
  acquire(&supermem.lock);
  r->next = supermem.freelist;
  supermem.freelist = r;
  release(&supermem.lock);
}
```

接下来，我们还需要为超级页添加实现分配的函数，总体的superpage的流程，大多都需要以普通页的分配释放映射为参考，而这里，其实就是kalloc换个皮：

```c
void* superalloc(void) {
  struct run *r;
  // 获取锁
  acquire(&supermem.lock);
  r = supermem.freelist;
  if(r)
    // 更新空闲页链表
    supermem.freelist = r->next;
  release(&supermem.lock);

  if(r)
    memset((char*)r, 5, SUPERPGSIZE); // fill with junk
  return (void*)r; 
}
```

这样，最底层需要被用到的函数就完成了，而sbrk在中间还会调用growproc，在其中调用vmmalloc为我们的进程分配内存，此处，就需要修改我们的uvmalloc，当需要的内存过大时，分配超级页，但是值得注意的一点是，我们分配过程中肯定需要进行内存对齐，由于超级页实在是太大了，肯定会非常浪费空间，所以我们应当先将对齐需要浪费的内存空间给利用起来。

```c
uint64
uvmalloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz, int xperm)
{
  char *mem;
  uint64 a; 
  int sz; 
    
  if(newsz < oldsz)
    return oldsz;
    
  oldsz = PGROUNDUP(oldsz);
  // 利用超级页对其所浪费的空间
  for(a = oldsz; a < SUPERPGROUNDUP(oldsz) && a < newsz; a += sz){
    sz = PGSIZE; 
    mem = kalloc(); 
    if(mem == 0){
      uvmdealloc(pagetable, a, oldsz); 
      return 0;
    }
#ifndef LAB_SYSCALL
    memset(mem, 0, sz);
#endif
    if(mappages(pagetable, a, sz, (uint64)mem, PTE_R|PTE_U|xperm) != 0){
      kfree(mem);
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
  }
  // 第二步：尽可能使用超级页批量分配，提升性能并减少页表开销
  // 之所以加上 a + SUPERPGSIZE < newsz 的条件，是为了尽可能少地浪费内存
  for(; a + SUPERPGSIZE < newsz; a += sz){
    sz = SUPERPGSIZE; 
    mem = superalloc(); 
    if(mem == 0){
      break;
    }
#ifndef LAB_SYSCALL
    memset(mem, 0, sz);
#endif
    if(mappages(pagetable, a, sz, (uint64)mem, PTE_R|PTE_U|xperm) != 0){
      superfree(mem);
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
  }
  // 第三步：对剩下不足一个超级页的部分用普通页补齐
  for(; a < newsz; a += sz){
    sz = PGSIZE;
    mem = kalloc();
    if(mem == 0){
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
#ifndef LAB_SYSCALL
    memset(mem, 0, sz);
#endif
    if(mappages(pagetable, a, sz, (uint64)mem, PTE_R|PTE_U|xperm) != 0){
      kfree(mem);
      uvmdealloc(pagetable, a, oldsz);
      return 0;
    }
  }
  // 返回新分配完成后的地址空间大小
  return newsz;
}
```

值得注意的是，uvmdealloc也需要被重新修改，现在当我们分配一个超级页之后，使用的mappages依旧是原来的样子，他现在只能做到对普通页进行映射，所以我们还需要去修改mappages，使得它能够映射超级页，与此同时，还需要一个superwalk函数：

```c
// 超级页映射只需要走到第1级，不需要走到最底层，这里也是我看别人博客才了解到的。
// 所以这里就没有必要用循环了。
pte_t* superwalk(pagetable_t pagetable, uint64 va, int alloc)
{
  // 检查虚拟地址是否越界
  if(va >= MAXVA)
    panic("superwalk");
  // 获取第2级页表中的页表项
  pte_t *pte = &pagetable[PX(2, va)];
  if(*pte & PTE_V) {
    // 如果该页表项有效（页表存在），进入第1级页表
    pagetable = (pagetable_t)PTE2PA(*pte); // 获取下一层页表的物理地址
    return &pagetable[PX(1, va)];          // 返回第1级页表中对应的 PTE 指针
  } else {
    // 页表项无效，页表不存在，需要根据 alloc 决定是否分配新的页表页
    // 注意：即使是映射超级页，它的页表结构也是由普通页组成的
    if(!alloc || (pagetable = (pde_t*)kalloc()) == 0)
      return 0;  // 分配失败或不允许分配，返回 NULL
    memset(pagetable, 0, PGSIZE); // 清零新分配的页表页
    *pte = PA2PTE(pagetable) | PTE_V; // 设置上层页表项为新分配页的地址，并置有效位
    // 返回第0级页表中的项
    return &pagetable[PX(0, va)];
  }
}




// - pagetable：进程页表
// - va：要映射的起始虚拟地址，必须页对齐
// - size：映射的大小，必须是页大小的倍数
// - pa：起始物理地址，决定是否使用超级页
// - perm：页表项权限位（如 PTE_W、PTE_U 等）
int
mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
  uint64 a, last;
  uint64 pgsize;
  pte_t *pte;

  // 根据物理地址是否超过 SUPERBASE 判断是否使用超级页
  if (pa >= SUPERBASE)
    pgsize = SUPERPGSIZE;
  else
    pgsize = PGSIZE; 

  // 检查虚拟地址是否页对齐
  if((va % pgsize) != 0)
    panic("mappages: va not aligned");

  // 检查 size 是否是页大小的整数倍
  if((size % pgsize) != 0)
    panic("mappages: size not aligned");

  if(size == 0)
    panic("mappages: size");

  a = va;
  last = va + size - pgsize;

  // 遍历每一页并映射
  for(;;){
    // 1. 根据页大小选择 walk 方式，获取/创建对应的页表项地址
    if(pgsize == PGSIZE && (pte = walk(pagetable, a, 1)) == 0)
      return -1;
    if(pgsize == SUPERPGSIZE && (pte = superwalk(pagetable, a, 1)) == 0)
      return -1;

    // 2. 如果已经存在映射，说明重复映射，报错
    if(*pte & PTE_V)
      panic("mappages: remap");

    // 3. 设置页表项，包含物理地址 + 权限 + 有效位
    *pte = PA2PTE(pa) | perm | PTE_V;

    // 4. 判断是否完成整个映射区间
    if(a == last)
      break;

    // 5. 前进到下一页
    a += pgsize;
    pa += pgsize;
  }

  return 0;
}

```

我们走到这一步，回到我们刚刚留下的一个问题，uvmdealloc用来处理释放内存，我们需要修改它，使得其支持超级页的释放，其中，uvmdealloc调用了uvmunmap函数，用于取消映射和释放页表，所以我们的核心就放在了他的上面：

```c
//   pagetable：进程的页表
//   va：要解除映射的起始虚拟地址
//   npages：要解除映射的页数
//   do_free：是否释放物理页
void uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free)
{
  uint64 a;
  pte_t *pte;
  int sz;
  // 检查虚拟地址必须页对齐
  if((va % PGSIZE) != 0)
    panic("uvmunmap: not aligned");
  // 遍历每一页，解除映射
  for(a = va; a < va + npages * PGSIZE; a += sz){
    sz = PGSIZE;
    // 获取页表项（不分配页表，所以 alloc=0）
    if((pte = walk(pagetable, a, 0)) == 0)
      panic("uvmunmap: walk");
    // 检查页表项是否有效（是否映射）
    if((*pte & PTE_V) == 0) {
      printf("va=%ld pte=%ld\n", a, *pte);
      panic("uvmunmap: not mapped");
    }
    // 检查该页表项是否为叶子节点（是否真正映射了物理页）
    if(PTE_FLAGS(*pte) == PTE_V)
      panic("uvmunmap: not a leaf");
    // 取得对应物理地址
    uint64 pa = PTE2PA(*pte);
    // 如果是超级页（物理地址 >= SUPERBASE），则说明是 2MB 映射
    // 需要额外调整步长，使得下一次循环跳过整个超级页范围
    if (pa >= SUPERBASE){
      a += SUPERPGSIZE;
      a -= sz; // 因为循环里还会加 sz（4KB），这里先减去以抵消
    }
    // 如果需要释放物理页帧
    if(do_free){
      uint64 pa = PTE2PA(*pte);
      if (pa >= SUPERBASE) 
        superfree((void*)pa); // 释放超级页
      else 
        kfree((void*)pa);     // 释放普通页
    }
    // 清空页表项
    *pte = 0;
  }
}
```

此时，我们很多事情都已经干完了，接下来，回到提示，看看我们少干了些什么，我们会发现，我们的fork的条件似乎并未满足，在fork的时候，我们是通过`uvmcopy`来复制页表，所以此时修改ivmcopy的逻辑，使其能够分配超级页。

```c
// 复制用户页表的内容：
// 把 old 页表中从 0 到 sz 的映射复制到 new 页表中。
// 如果对应的是超级页则使用 superalloc，否则使用 kalloc。
// 复制时连同数据内容一起复制（即数据页克隆）。
// 返回 0 表示成功，-1 表示失败（并清理 new 页表中已分配的页）。
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;
  uint flags;
  char *mem;
  int szinc;

  // 遍历 old 页表中从 0 到 sz 的所有虚拟地址
  for(i = 0; i < sz; i += szinc){
    szinc = PGSIZE;
    // 找到 old 页表中当前虚拟地址 i 对应的页表项
    // 由于此时使用的页已经被正确映射
    // 所以普通的walk也可以找到超级页的pte
    if((pte = walk(old, i, 0)) == 0)
      panic("uvmcopy: pte should exist");
    if((*pte & PTE_V) == 0)
      panic("uvmcopy: page not present");
    // 提取物理地址与标志位
    pa = PTE2PA(*pte);
    flags = PTE_FLAGS(*pte);
    // 判断是普通页还是超级页
    if(pa >= SUPERBASE) {
      // 超级页大小设置为 2MB
      szinc = SUPERPGSIZE;
      // 分配新的超级页
      if((mem = superalloc()) == 0)
        goto err;
    }
    // 普通页
    else if((mem = kalloc()) == 0)
      goto err;
    // 拷贝数据内容（从旧物理页拷贝到新分配的物理页）
    memmove(mem, (char*)pa, szinc);
    // 映射到新页表中
    if(mappages(new, i, szinc, (uint64)mem, flags) != 0){
      if(szinc == SUPERPGSIZE)
        superfree(mem);
      else
        kfree(mem);
      goto err;
    }
  }
  return 0;
 err:
  // 若出错，释放 new 页表中已分配的页
  uvmunmap(new, 0, i / PGSIZE, 1);
  return -1;
}
```

至此，完成。。。

这个超级页的实验还是很难的，虽然根据提示我也能做出来一些，但是我一个人还是没办法完成的，总结一下：

超级页的实验主要是迫使我们去真正的理解普通的页是如何分配的，如何映射，如何释放的，然后让我们自己去重新实现一遍逻辑，对于超级页而言，我觉得难想的就是映射，我还以为也要映射三级，但是事实上根本不用，这就是理解上的问题了，这个实验去做一遍还是很不错的，虽然也参考了别人的博客，但总归是自己敲了一遍，接下来几天就要all in 期中复习了，虽然想继续下一个课程，但是只能先暂缓了。

参考文献：

[[MIT 6.1810\]Lab3 page table](https://erlsrnby04.github.io/2024/10/05/MIT-6-1810-Lab3-page-tables/)。

