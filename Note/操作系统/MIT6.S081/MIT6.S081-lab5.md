# MIT6.S081-lab5

## 1. Copy On Write

这部分我学了3天，两天在死磕这个lab，总算是写出来了，不得不说，过程中确实需要去大量地阅读源码，需要去理解每一步，最后才可以完成这个lab，如果理解了COW的机制，实际上这个lab是不难的。下面开始！

cow是什么？就是写时复制，根据hint，我们需要为每一个页创建一个引用的字段，以此来表示他们被多少proc引用，方便在释放页的时候，只有在我们的页引用数变为了0，才会真正的回收，在这里，我们会用一个全局的数组来表示这个引用数。而其他情况，则会减少引用的数量。

同时，我们还需要去在usertrap里面判断我们的页错误引发的trap，这个很简单就能够判断，重点是做写时复制的处理。

另外，根据hint，我们会知道，我们需要修改copyout这个函数，以此来应付我们需要在内核空间向用户空间写入数据的情况。

至此，我们的大体框架已经搭建起来，我们需要在defs里面定义一下我们需要创建的函数：

```c
void refdown(void* pa);
void refup(void* pa);
uint64 refidx(uint64 pa);
void* copyPA(void* pa);  // 真正的复制物理页 
void copyonwrite(pagetable_t pagetable, uint64 va);  // 写时复制逻辑
int iscowpage(pagetable_t pagetable, uint64 va);  // 判断是否为写时复制页
```

然后在kalloc.c里面，定义我们的锁(应对并发)和全局数组，这里锁还没有讲，但是不影响。

同时，我们需要在kinit里面初始化我们的锁。

```c
struct spinlock ref_lock;
int pm_ref[(PHYSTOP - KERNBASE) / PGSIZE]; 

void
kinit()
{
  initlock(&kmem.lock, "kmem");
  initlock(&ref_lock, "pm_ref");
  freerange(end, (void*)PHYSTOP);
}
```

剩下的都在kalloc.c里面的，一次性给出来，这个文件基本都修改了。

```c
// 释放一页物理内存
void
kfree(void *pa)
{
  struct run *r;

  // 检查地址是否合法：
  // - 必须页对齐
  // - 地址不能小于 end（内核代码和数据结束处）
  // - 地址不能超过物理内存最大值 PHYSTOP
  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  acquire(&ref_lock);

  pm_ref[refidx((uint64)pa)] --;

  // 如果引用计数已经为 0，说明没人再使用这页，可以释放
  if(pm_ref[refidx((uint64)pa)] <= 0){
    memset(pa, 1, PGSIZE);
    r = (struct run*)pa;

    acquire(&kmem.lock);
    r->next = kmem.freelist;
    kmem.freelist = r;
    release(&kmem.lock);
  }

  release(&ref_lock);
}

void *
kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r)
    kmem.freelist = r->next;
  release(&kmem.lock);

  if(r){
    memset((char*)r, 5, PGSIZE);

    // 初始化引用计数为 1（即当前只有一个使用者）
    pm_ref[refidx((uint64)r)] = 1;
  }
  return (void*)r;
}

// 将物理地址转换为引用计数数组的索引
uint64
refidx(uint64 pa){
  return (pa - KERNBASE) / PGSIZE;
}

// 增加某页的引用计数（例如共享页时）
void
refup(void* pa){
  acquire(&ref_lock);
  pm_ref[refidx((uint64)pa)] ++;
  release(&ref_lock);
}

// 减少某页的引用计数（不是释放，只是标记减少）
void
refdown(void* pa){
  acquire(&ref_lock);
  pm_ref[refidx((uint64)pa)] --;
  release(&ref_lock);
}

// 写时复制的物理页复制函数
// 如果引用计数 > 1，就分配一页新的内存并拷贝数据
// 否则直接返回原始页（不用复制）
void*
copyPA(void* pa){
  acquire(&ref_lock);

  // 如果引用计数只有 1，说明没有其他用户，可以直接写
  if(pm_ref[refidx((uint64)pa)] <= 1){
    release(&ref_lock);
    return pa;
  }

  // 否则分配新页（进行复制）
  char* new = kalloc();
  if(new == 0){
    release(&ref_lock);
    panic("out of memory");
    return 0;
  }

  // 将原页内容复制到新页
  memmove((void*)new, pa, PGSIZE);

  // 原页的引用计数减一（当前页会换成新页）
  pm_ref[refidx((uint64)pa)] --;

  release(&ref_lock);

  return (void*)new;
}
```

随后，我们直接到trap.c去修改我们的usertrap的判断逻辑：

```c
  if(r_scause() == 8){
    // system call

    if(killed(p))
      exit(-1);

    // sepc points to the ecall instruction,
    // but we want to return to the next instruction.
    p->trapframe->epc += 4;

    // an interrupt will change sepc, scause, and sstatus,
    // so enable only now that we're done with those registers.
    intr_on();

    syscall();
  } else if((which_dev = devintr()) != 0){
    // ok
  } else if((r_scause() == 15 || r_scause() == 13) && iscowpage(p->pagetable, r_stval())){   // 加在这里！！
    copyonwrite(p->pagetable, r_stval());															  // 这里是修改的部分
  } else {
    printf("usertrap(): unexpected scause 0x%lx pid=%d\n", r_scause(), p->pid);
    printf("            sepc=0x%lx stval=0x%lx\n", r_sepc(), r_stval());
    setkilled(p);
  }

  if(killed(p))
    exit(-1);

  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2)
    yield();
  usertrapret();
}
```

我们此处实现了写时复制页判断逻辑，随后需要去实现我们的写时复制的拷贝逻辑：

kernel/vm.c

```c
// 处理写时复制异常，将虚拟地址 va 映射的物理页复制一份并映射为可写
void
copyonwrite(pagetable_t pagetable, uint64 va) {
  // 将地址向下对齐到页边界（因为页表管理的是页对齐地址）
  va = PGROUNDDOWN((uint64)va);

  // 查找虚拟地址对应的页表项
  pte_t* pte = walk(pagetable, va, 0);
  // 从页表项中提取物理地址（即原始共享页）
  uint64 pa = PTE2PA(*pte);

  // 调用 copyPA 复制物理页，若引用计数 > 1 就复制，否则返回原始页
  void* new = copyPA((void*)pa);
  if((uint64)new == 0){
    panic("cowcopy_pa err\n");
    exit(-1);
  }

  // 设置新的页表项权限：
  // - 添加写权限（PTE_W）
  // - 去掉 COW 标记位（PTE_COW）
  uint64 flags = (PTE_FLAGS(*pte) | PTE_W) & (~PTE_COW);

  // 解除旧的页映射（不回收物理内存）
  uvmunmap(pagetable, va, 1, 0);

  // 将新的物理页映射到相同的虚拟地址，带上正确的权限
  if(mappages(pagetable, va, PGSIZE, (uint64)new, flags) == -1){
    kfree(new);
    panic("cow mappages failed");
  }
}
```

根据hint，当我们执行fork的时候，我们会调用uvmcopy这个函数来执行页的复制，此处，为了实现我们的cow，我们需要对他进行修改，来保证我们可以正确的得到一个共享页，此处就不需要真正的去复制页了，而仅仅是修改一下权限位，随后映射即可。

```c
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;
  uint flags;

  for(i = 0; i < sz; i += PGSIZE){
    if((pte = walk(old, i, 0)) == 0)
      panic("uvmcopy: pte should exist");
    if((*pte & PTE_V) == 0)
      panic("uvmcopy: page not present");
    pa = PTE2PA(*pte);
	 // 修改权限位
    if(*pte & PTE_W){
        *pte &= ~PTE_W;
        *pte |= PTE_COW;
    }
    flags = PTE_FLAGS(*pte);
    // 映射
    if(mappages(new, i, PGSIZE, (uint64)pa, flags) != 0){
      goto err;
    }
    refup((void*)pa);
  }
  return 0;

 err:
  uvmunmap(new, 0, i / PGSIZE, 1);
  return -1;
}
```

最后，我们还需要将copyout进行修改，这里的copyout实际上就是将一些数据拷贝到用户空间，比如read这个系统调用，就会调用copywrite，将读取指定的数据**写入**到用户空间，此时，我们需要进行判断他是否为写时复制页，并且之后去判断它的权限位是否正确，是否能够进行写入，是否有效等等，最后，我们才可以执行写入操作：

```c
int
copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len)
{
  uint64 n, va0, pa0;
  pte_t *pte;

  while(len > 0){
    // 先进行页面对其
    // printf("dst:%ld ", dstva);
    va0 = PGROUNDDOWN(dstva);
    // printf("va:%ld ", va0);
    if(va0 >= MAXVA)
      return -1;
    // 判断写时复制页
    if(iscowpage(pagetable, va0)) {
      copyonwrite(pagetable, va0);
    }
    // walk，看看最后的权限位是否正确。
    pte = walk(pagetable, va0, 0);
    if(pte == 0 || (*pte & PTE_V) == 0 || (*pte & PTE_U) == 0 || (*pte & PTE_W) == 0)
      return -1;
	 // 获取物理地址
    pa0 = PTE2PA(*pte);
	 // 下面不需要修改了
    n = PGSIZE - (dstva - va0);
    if(n > len)
      n = len;
    memmove((void *)(pa0 + (dstva - va0)), src, n);

    len -= n;
    src += n;
    dstva = va0 + PGSIZE;
  }
  return 0;
}
```

这样，我们的lab5就完成了。

测试结果：

```bash
== Test running cowtest == 
$ make qemu-gdb
(27.4s) 
== Test   simple == 
  simple: OK 
== Test   three == 
  three: OK 
== Test   file == 
  file: OK 
== Test   forkfork == 
  forkfork: OK 
== Test usertests == 
$ make qemu-gdb
(97.9s) 
== Test   usertests: copyin == 
  usertests: copyin: OK 
== Test   usertests: copyout == 
  usertests: copyout: OK 
== Test   usertests: all tests == 
  usertests: all tests: OK 
```

---

说难也不算难，简单倒也不简单，重要的是理解xv6的运作。
