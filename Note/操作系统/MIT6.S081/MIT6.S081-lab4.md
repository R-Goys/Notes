# MIT6.S081-lab4



注：本篇lab的前置知识在《MIT6.S081-lab3前置》

## 1. RISC-V assembly

第一个问题

> Which registers contain arguments to functions? For example, which register holds 13 in main's call to `printf`?

我们先来看看main干了什么：

```assembly
void main(void) {
  1c:	1141                	addi	sp,sp,-16
  1e:	e406                	sd	ra,8(sp) 
  20:	e022                	sd	s0,0(sp)
  22:	0800                	addi	s0,sp,16
  printf("%d %d\n", f(8)+1, 13);							# 编译器直接算出来了，无需调用f和g函数
  24:	4635                	li	a2,13              	# printf参数存入寄存器a2
  26:	45b1                	li	a1,12             
  28:	00001517          	auipc	a0,0x1        	 	# 存入格式格式字符串的大致地址，printf的第一个参数
  2c:	84850513          	addi	a0,a0,-1976        	# a0 = a0 - 1976，即精确地得到格式字符串地址 "%d %d\n"
  30:	68c000ef          	jal	6bc <printf>
  exit(0);
  34:	4501                	li	a0,0 
  36:	26e000ef          	jal	2a4 <exit>
```

综上，a0,a1,a2存放了对应的调用函数所要用的参数。

第二个问题

> is the call to function `f` in the assembly code for main? Where is the call to `g`? (Hint: the compiler may inline functions.)

对f的调用我发现已经被编译器所优化了，这里直接将一个立即数存入了a1中：

```assembly
  26:	45b1                	li	a1,12
```

第三个问题

> At what address is the function `printf` located?

根据汇编代码，我们可以知道，printf位于`6bc`处，事实上，我们可以在call.asm里面搜索printf，我们可以找到，函数的入口确实是`6bc`：

```assembly
....
void
printf(const char *fmt, ...)
{
 6bc:	711d                	addi	sp,sp,-96
 6be:	ec06                	sd	ra,24(sp)
 6c0:	e822                	sd	s0,16(sp)
 6c2:	1000                	addi	s0,sp,32
 .....
```

第四个问题

> What value is in the register `ra` just after the `jalr` to `printf` in `main`?

在main里面，我们很容易发现，根本没有用到ra寄存器，但是其实，ra存储的一般是我们的函数返回的地址，所以在我们调用jal的时候， 会自动将下一条指令的地址存入ra寄存器中，即`0x34`

第五个问题

> Run the following code. What is the output?
>
> ```c
> 	unsigned int i = 0x00646c72;
> 	printf("H%x Wo%s", 57616, (char *) &i);
> ```

输出：`He110 World`，大端模式则需要将i修改为`0x72 6c 64 00`，我们可以发现就是反转了一下，而另一个数字无需修改，因为这个打印的是16进制表示数字，与大端小端字节序无关。

第六个问题

> In the following code, what is going to be printed after `'y='`? (note: the answer is not a specific value.) Why does this happen?
>
> ```c
> printf("x=%d y=%d", 3);
> ```

未定义行为，这取决于对应寄存器的值。

## 2. Backtrace

常爆panic的同学应该会对这个backtrace非常熟悉，他会打印我们函数调用链路的函数返回的地方。这就是我们实验需要实现的东西了，根据lab1，我们可以知道，函数调用的时候，都会把函数返回的地方的地址存储起来，那么我们的目的，就是找到这个存储地址的地方，并且将他打印出来。

难点就在于怎么去找到这个地址，光靠自己去推理，肯定是很困难的，这时候就需要看给我们的hint。

首先，我们向kernel/defs.h中添加`static inline uint64 r_fp()`这个函数，用来在我们的当前需要编写的backtrace中获取当前的帧指针，以此为基础，来获取之前的函数返回地址。

随后继续往下看：

> - These [lecture notes](https://pdos.csail.mit.edu/6.1810/2023/lec/l-riscv.txt) have a picture of the layout of stack frames. Note that the return address lives at a fixed offset (-8) from the frame pointer of a stackframe, and that the saved frame pointer lives at fixed offset (-16) from the frame pointer.
> - Your `backtrace()` will need a way to recognize that it has seen the last stack frame, and should stop. A useful fact is that the memory allocated for each kernel stack consists of a single page-aligned page, so that all the stack frames for a given stack are on the same page. You can use `PGROUNDDOWN(fp)` (see `kernel/riscv.h`) to identify the page that a frame pointer refers to.

我们可以通过这个hint知道，我们保存的地址的偏移量是-8，而想要得到上一个帧地址，就需要-16，然后继续以此为-8为偏移量去得到我们的保存的return地址，并且在遇到页的边缘的时候，我们就会停止回溯。

于是，我们的backtrace代码就可以写出来了：

```c
void backtrace(void) {
  printf("backtrace:\n");
  uint64 ra, fp = r_fp();

  // 获取前一个帧指针的位置，位于当前帧指针 fp - 16 的位置
  // 按照调用约定，fp-8 是返回地址，fp-16 是上一个函数的帧指针
  uint64 pre_fp = *((uint64*)(fp - 16));

  // 当上一个帧指针和当前帧指针还在同一个物理页中（即没有越过页边界）时，继续回溯
  while (PGROUNDDOWN(fp) == PGROUNDDOWN(pre_fp)) {

    ra = *(uint64 *)(fp - 8);

    printf("%p\n", (void*)ra);
    // 更新当前帧指针为上一个帧指针
    fp = pre_fp;
    // 继续获取上一个帧的帧指针
    pre_fp = *((uint64*)(fp - 16));
  }

  // 打印最后一个返回地址（最后一个栈帧）
  ra = *(uint64 *)(fp - 8);
  printf("%p\n", (void*)ra);
}
```

除此之外，记得在kernel/defs.h定义我们的backtrace函数，并且将这个函数添加到sys_sleep中。

这样，backtrace就算完成了。

## 3. Alarm

实验要求是注册一个时间间隔和函数到当前的cpu，到点的时候就会调用这个函数，并且期间要求恢复我们的当前进程的上下文(寄存器)不受影响，简单来讲，就是一个非常tiny的trap。

首先我们阅读hint，这个实验不读hint真的是没法做。

> - You'll need to modify the Makefile to cause `alarmtest.c` to be compiled as an xv6 user program.
>
> - The right declarations to put in user/user.h are:
>
>   ```
>       int sigalarm(int ticks, void (*handler)());
>       int sigreturn(void);
>   ```
>
> - Update user/usys.pl (which generates user/usys.S), kernel/syscall.h, and kernel/syscall.c to allow `alarmtest` to invoke the sigalarm and sigreturn system calls.
>
> - For now, your `sys_sigreturn` should just return zero.
>
> - Your `sys_sigalarm()` should store the alarm interval and the pointer to the handler function in new fields in the `proc` structure (in `kernel/proc.h`).
>
> - You'll need to keep track of how many ticks have passed since the last call (or are left until the next call) to a process's alarm handler; you'll need a new field in `struct proc` for this too. You can initialize `proc` fields in `allocproc()` in `proc.c`.
>
> - Every tick, the hardware clock forces an interrupt, which is handled in `usertrap()` in `kernel/trap.c`.
>
> - You only want to manipulate a process's alarm ticks if there's a timer interrupt; you want something like
>
>   ```
>       if(which_dev == 2) ...
>   ```
>
> - Only invoke the alarm function if the process has a timer outstanding. Note that the address of the user's alarm function might be 0 (e.g., in user/alarmtest.asm, `periodic` is at address 0).
>
> - You'll need to modify `usertrap()` so that when a process's alarm interval expires, the user process executes the handler function. When a trap on the RISC-V returns to user space, what determines the instruction address at which user-space code resumes execution?
>
> - It will be easier to look at traps with gdb if you tell qemu to use only one CPU, which you can do by running
>
>   ```
>       make CPUS=1 qemu-gdb
>   ```
>
> - You've succeeded if alarmtest prints "alarm!".

> - Your solution will require you to save and restore registers---what registers do you need to save and restore to resume the interrupted code correctly? (Hint: it will be many).
> - Have `usertrap` save enough state in `struct proc` when the timer goes off that `sigreturn` can correctly return to the interrupted user code.
> - Prevent re-entrant calls to the handler----if a handler hasn't returned yet, the kernel shouldn't call it again. `test2` tests this.
> - Make sure to restore a0. `sigreturn` is a system call, and its return value is stored in a0.

这些hint可谓是信息量很大了，简单梳理一下，我们先将需要的系统调用框架先搭好：

makefile

```bash
UPROGS=\
	$U/_cat\
	$U/_echo\
	$U/_forktest\
	$U/_grep\
	$U/_init\
	$U/_kill\
	$U/_ln\
	$U/_ls\
	$U/_mkdir\
	$U/_rm\
	$U/_sh\
	$U/_stressfs\
	$U/_usertests\
	$U/_grind\
	$U/_wc\
	$U/_zombie\
	// 添加这一行
	$U/_alarmtest\
```

user/usys.pl

```perl
entry("sigalarm");
entry("sigreturn");
```

user/user.h

```c
// lab
int sigalarm(int ticks, void (*handler)());
int sigreturn(void);
```

kernel/syscall.h

```c
#define SYS_sigalarm 22
#define SYS_sigreturn 23
```

kernel/syscall.c

```c
[SYS_sigalarm] sys_sigalarm,
[SYS_sigreturn] sys_sigreturn
// 这部分加在数组里面，做过之前的lab懂得都懂
```

目前我们大体的框架是弄好了，随后着手去看我们的hint，我们可以知道，如果发生了定时器中断，我们的which_dev就是2，hint告诉了我们这一点，于是，我们可以在这一部分代码块写下我们的中断逻辑，但是这部分应该如何去写呢？我们需要去执行我们之前注册的函数，并且需要保存当前的trapframe，保证之后还能够回到这里，并且还需要去判断计时器的时间，并且做一些加减操作，所以，我们在此之前，还需要对我们的proc结构体进行一些修改：

kernel/proc.h

```c
//为proc结构体添加以下字段
  uint64 interval;              // 间隔
  void (*handler)();            // 定时处理的函数
  uint64 ticks;                 // 上一次调用函数距离的时间
  struct trapframe *alarm_trapframe;  // 用于恢复 trapframe
  int alarm_goingoff;           // 是否正在alarm，防止嵌套的中断，导致trapframe丢失
```

我们既然多了这么多字段，那么必须要在allocproc里面，也为这些字段进行初始化

```c
static struct proc*
allocproc(void)
{
  //...

found:
  //...

  if((p->alarm_trapframe = (struct trapframe *)kalloc()) == 0) {
    freeproc(p);
    release(&p->lock);
    return 0;
  }
  p->ticks = 0;
  p->handler = 0;
  p->interval = 0;
  p->alarm_goingoff = 0;

  //...

  return p;
}
```

同时，在释放proc的时候，也需要执行对应的操作：

```c
static void
freeproc(struct proc *p)
{
  //...
  // free alarm trapframe
  if(p->alarm_trapframe)
    kfree((void*)p->alarm_trapframe);
  p->alarm_trapframe = 0;
  //...

  p->ticks = 0;
  p->handler = 0;
  p->interval = 0;
  p->alarm_goingoff = 0;
  p->state = UNUSED;
}
```

随后，我们需要去编写我们的具体的系统调用的逻辑，sigalarm和sigreturn

```c
uint64
sys_sigalarm(void) {
  int n;
  uint64 handler;
  // 获取参数
  argint(0, &n);
  argaddr(1, &handler);
  // 调用下一层
  return sigalarm(n, (void(*)())(handler));
}

uint64
sys_sigreturn(void) {
  return sigreturn();
}
```

我们的sigreturn和sigalarm定义在trap.c

```c
int sigalarm(int ticks, void(*handler)()) {
  // 初始化alarm
  struct proc *p = myproc();
  p->interval = ticks;
  p->handler = handler;
  p->ticks = 0;
  return 0; 
}

int sigreturn() {
  struct proc *p = myproc();
  // 恢复之前的trapframe，并清除alarm标志位
  *(p->trapframe) = *(p->alarm_trapframe);

  p->alarm_goingoff = 0;
  // 这里返回a0的原因是，当我们执行return的时候，返回值会被保存在a0中
  // 导致a0被覆盖，所以此时直接返回a0即可，我们在最后会进行分析
  return p->trapframe->a0;
}
```

当然，这两个函数还需要在kernel/defs.h中声明，否则会报错！

最后，回到我们的usertrap函数，我们会在这里完成最后的工作

```c
void
usertrap(void)
{
  //...
  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2) {
    if(p->interval != 0) { // 如果设定了时钟事件
      if(p->ticks++ == p->interval) {
        if(!p->alarm_goingoff) { // 确保没有时钟正在运行
          p->ticks = 0;
          *(p->alarm_trapframe) = *(p->trapframe);
          p->trapframe->epc = (uint64)p->handler;
          p->alarm_goingoff = 1;
        }
      }
    }
    yield();
  }
  usertrapret();
}
```

我们在which_dev满足等于2的条件的时候，会增加我们的时钟计时，当达到我们的间隔时间，就会保存我们的trapframe，并且修改我们的epc，epc是什么？就是我们返回用户态的时候，会执行的代码的指针，我们将需要执行的函数的地址赋给epc，也就是说，我们接下来就会去执行它，当然，如果需要我们的之前执行的函数能够恢复，也就意味着，我们需要在注册的函数里面主动去调用sigreturn，然后才能恢复到我们原来的用户态的中断的地方，这样，就完成了这个系统调用的闭环。

回到刚刚的问题，为什么要返回a0?

我们可以查看汇编代码来解决这个问题

kernel/kernel.asm

```assembly
  return p->trapframe->a0;
    80001c44:	6d3c                	ld	a5,88(a0)      # 加载 p->trapframe 的地址到 a5，偏移 88 字节是 trapframe*
}
    80001c46:	5ba8                	lw	a0,112(a5)     # 加载 trapframe->a0 的值到 a0，偏移 112 字节是 a0 寄存器的位置
    80001c48:	60a2                	ld	ra,8(sp)       # 恢复调用者的返回地址（ra）
    80001c4a:	6402                	ld	s0,0(sp)       # 恢复调用者的帧指针（s0）
    80001c4c:	0141                	addi	sp,sp,16       # 恢复栈指针（释放本函数栈帧）
    80001c4e:	8082                	ret              # 返回到调用者，返回值已保存在 a0 中
```

我们可以看见，我们会将返回的代码赋给a0，但是即便如此，我们的a5也会被覆盖，所以最好的办法还是自己用汇编来实现这些上下文的切换。

那么最后，我们的alarm实验就完成了。

```bash
== Test backtrace test == 
$ make qemu-gdb
backtrace test: OK (2.6s) 
== Test running alarmtest == 
$ make qemu-gdb
(4.8s) 
== Test   alarmtest: test0 == 
  alarmtest: test0: OK 
== Test   alarmtest: test1 == 
  alarmtest: test1: OK 
== Test   alarmtest: test2 == 
  alarmtest: test2: OK 
== Test   alarmtest: test3 == 
  alarmtest: test3: OK 
== Test usertests == 
$ make qemu-gdb
usertests: OK (151.6s) 
```

即便之前读过了系统调用陷入的一系列代码，通过写这个lab4的实验，也是比较困难的，但也能学到一些东西的，虽然中途确实看了别人的代码，但是总归是写出来的，重要的不是看了别人的多少的代码，我倒是觉得这并不可耻，在一些无聊的地方卡住好几个小时没有一点进展，而因为秉持着学术诚信最后却因为一些bug而放弃，这反倒是我最不想看到的，最重要的是从这个实验中学到了多少，所以，在这里，我将自己学到的分享出去，希望能够帮助更多的人。

参考文献：

[miigon'blog](https://blog.miigon.net/)
