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

