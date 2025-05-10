# MIT6.S081-lab1

注：实验环境在我的汇编随手记的末尾部分有搭建教程。

## 0.前置

### 第零章

xv6为我们提供了多种系统调用，其中，exec将从某个文件里读取内存**镜像**(这确实是一个好的说法)，并且将其替换到调用它的内存空间，也就是这个打开的文件(一切皆文件)替换了当前的进程，exec仅仅是这样的功能，同时，执行完成之后，exec并不会返回当前的调用进程，而是执行我们已经加载好的指令！

如果你阅读过手写docker或者类似讲过相关概念的书，你一定会知道，我们执行命令事实上是通过创建一个子进程，再在子进程中exec我们需要的命令！

exec**并不是**执行程序这个操作的全部，而**只是**将当前进程替换为某个可执行文件的**工具**，它需要结合 fork 使用，才是完整的执行命令的流程。

而命令执行完成，我们的子进程就会调用exit，使得我们的父进程从wait中返回。

**文件描述符是啥？**一切皆文件，我们的文件描述符可以是管道，文件，目录，socket的抽象，但是，值得注意的是，文件描述符并不代表了这个文件，而是指向这个文件的"指针"，使得我们可以对其进行访问，我们可以获取多个指向同一个文件的文件描述符，并且能够对其进行写入，读取操作。

应用比如cat指令，cat并不关心你的文件描述符指向的是什么，使得我们可以轻松的实现cat指令，所以文件描述符是一个很棒的抽象。

甚至，fork和文件描述符可以实现我们的重定向，比如当我们fork一个进程之后，关闭子进程的文件描述符0(标准输入)，然后重新打开一个我们指定的文件，文件描述符0指向的是我们指定的文件，也就是说，我们的标准输入来自于文件，而不是键盘了！然后我们执行cat，就会打印出我们的文件内容，指令为：`cat < test.txt`。

一般来说，通过dup和fork产生的文件描述符都将共享同一个偏移量，但是有一些特殊情况，这里不详细说了。

**管道**

这段代码值得分析，先创建一个管道，读端文件描述符为p[0]，写端为p[1]，在我们的子进程中，先将文件描述符0(标准输入)关闭，然后调用我们的dup将文件描述符p[0]复制到标准输入中，此时，我们的wc就可以从文件中读取数据了，然后，我们还需要子进程的写端，因为子进程中，写端是无用的，如果不关闭，我们在wc进程中的read将会阻塞，无法返回。

而在父进程中，指向写入，然后关闭就行了。

```c
int p[2];
char *argv[2];
argv[0] = "wc";
argv[1] = 0;
pipe(p);
if(fork() == 0) {
    close(0);
    dup(p[0]);
    close(p[0]);
    close(p[1]);
    exec("/bin/wc", argv);
} else {
    write(p[1], "hello world\n", 12);
    close(p[0]);
    close(p[1]);
}
```

管道比临时文件强大得多，管道支持自动销毁，支持发送任意长度的数据，支持同步地进程间通信。

**文件系统**

我看这部分主要讲的是文件就是一棵树，前面没啥好说的

`mknod`表示创建一个设备文件，其元信息标志他是一个设备，并且记录了主设备号和辅设备号，他们确定了唯一设备，当进程打开这个文件的时候，内核会将读写操作**转发**到相应的设备上，而不是文件系统。

`fstat`可以通过文件描述符获取他所指向的文件的信息。

这里的一个概念也挺有意思的，就是文件名和文件有很大的区别，一个文件可以有多个文件名，一个文件名同一时刻指向一个文件(inode)，比如说下面：

```C
open("a", O_CREATE|O_WRONGLY);
link("a", "b");
```

这里创建了一个文件，然后通过link使得这个文件既叫a，又叫b，但是，此时我们如果执行`unlink('a')`，我们的inode和磁盘空间并不会被清空，因为此时我们的文件名b还指向它，所以一个文件的的inode和磁盘空间只有link数量为0的时候才会被清除

所以

```C
fd = open("/tmp/xyz", O_CREATE|O_RDWR);
unlink("/tmp/xyz");
```

是创建一个临时inode的最好方式。

## 1. Sleep

挺简单的，应该就是让我们提升自信心的，先fork一个子进程，在子进程中调用sleep，父进程等待。

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(1, "Usage: sleep seconds\n");
        exit(1);
    }   
    int pid = fork();
    if (pid == 0) {
        unsigned int seconds = atoi(argv[1]);
        sleep(seconds * 10);
        exit(0);
    } else {
        wait(0);
    }
    exit(0);
}
```

---

## 2. PingPong

大部分都是前置内容，也就是教材里面讲过的，需要创建两个管道，以供来相互通信，注意关闭读写端的时机：

```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
    int p1[2];
    int p2[2];
    pipe(p1);
    pipe(p2);
    int pid1 = fork();
    if (pid1 == 0) {
        close(p1[1]);
        close(p2[0]);
        char buf[1];
        read(p1[0], buf, 1);
        close(p1[0]);
        printf("%d: received ping\n", getpid());
        write(p2[1], "x", 1);
        close(p2[1]);
        exit(0);
    }
    int pid2 = fork();
    if (pid2 == 0) {
        close(p1[0]);
        close(p2[1]);
        write(p1[1], "x", 1);
        close(p1[1]);
        char buf[1];
        read(p2[0], buf, 1);
        close(p2[0]);
        printf("%d: received pong\n", getpid());
        write(p1[1], "x", 1);
        close(p1[1]);
        exit(0);
    }
}
```

## 3. Primes

我去，这个lab真牛逼，最核心的点就是dup去复用我们的管道，让管道可以及时地被释放，这真的很重要！否则你的程序大概率只能跑到40左右的数字(血的教训)，另外就是实验要求使用埃拉托色尼筛法，这一点我最开始也搞不懂要怎么去在管道之间传递这个数字，其实就是pipe不熟悉，还是问了gpt才明白，可以一个一个传，然后一个一个读取。

然后dup的使用也是参考了别人的blog，感觉自己就是菜。

总之感觉还是挺神奇的。

```c
#include "kernel/types.h"
#include "user/user.h"

void primes(int p0[2]) __attribute__((noreturn));

int main(int argc, char *argv[]) {
    int p[2];
    pipe(p);
    int pid = fork();
    if (pid == 0) {
        //管道的关闭逻辑在primes函数中
        primes(p);
    } else {
        close(p[0]);
        for (int i = 2; i <= 280; i++) {
            write(p[1], &i, sizeof(i));
        }
        close(p[1]);
        wait(0);
    }
    exit(0);
}


void primes(int old_pipe[2]) {
    //及时释放管道
    close(0);
    dup(old_pipe[0]);
    close(old_pipe[0]);
    close(old_pipe[1]);


    int prime;
    if (read(0, &prime, sizeof(prime)) == 0) {
        close(0);
        exit(0);
    }
    printf("prime %d\n", prime);
    //新建管道，并fork子进程
    int new_pipe[2];
    pipe(new_pipe);
    int pid = fork();
    if (pid == 0) {
        primes(new_pipe);
    } else {
        close(new_pipe[0]);
        int num;
        while (read(0, &num, sizeof(num))) {
            if (num % prime != 0) {
                write(new_pipe[1], &num, sizeof(num));
            }
        }
        close(0);
        close(new_pipe[1]);
        wait(0);
    }
    exit(0);
}
```

---

## 4. Find

实验hint，让我们可以从ls.c中知道怎么才可以展开当前目录，这部分完全是参考了ls.c里面的方法，知道了这一点，我们就很好做判断了。

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

void find(char *path, char *filename);

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(2, "Usage: find filename with path\n");
        exit(1);
    }
    //递归搜索
    find(argv[1], argv[2]);
    exit(0);
    
}

void find(char *path, char *filename) {
    char buf[512], *p;
    int fd;
    struct dirent de;
    struct stat st;

    if ((fd = open(path, 0)) < 0) {
        fprintf(2, "find: cannot open %s\n", path);
        return;
    }
    if (fstat(fd, &st) < 0) {
        fprintf(2, "find: cannot stat %s\n", path);
        close(fd);
        return;
    }
    switch (st.type) {
        case T_DIR:
            if (strlen(path) + 1 + DIRSIZ + 1 >= sizeof(buf)) {
                printf("find: path too long\n");
                break;
            }
            strcpy(buf, path);
            p = buf + strlen(buf);
            *p++ = '/';
            while (read(fd, &de, sizeof(de)) == sizeof(de)) {
                if (de.inum == 0)
                    continue;
                memmove(p, de.name, DIRSIZ);
                p[DIRSIZ] = 0;
                if (stat(buf, &st) < 0) {
                    printf("find: cannot stat %s\n", buf);
                    continue;
                }
                if (st.type == T_FILE && strcmp(de.name, filename) == 0) {
                    printf("%s\n", buf);
                }
                if (st.type == T_DIR && strcmp(de.name, ".") != 0 && strcmp(de.name, "..") != 0) {
                    find(buf, filename);
                }
            }
            break;
        default:
            if (strcmp(path, filename) == 0) {
                printf("%s\n", path);
            }
        }
    close(fd);
}
```

这里有一个很有意思很有意思的东西，我直接跳转到read的实现，实际上但是他会直接跳到qemu的文件里面，导致我以为我们的read是qemu封装好的，但是实际上并不是，read确确实实我们的xv6自己实现的！我们可以通过这样去追溯它的根源：

> 1. 在/user/usys.S中，找到有关read的字段，可以看见，它调用了SYS_read。
> 2. 回到/kernel/syscall.c，我们可以看见syscall_read的具体定义。
> 3. 跳转，我们会发现，调用了fileread这个函数，继续跳转
> 4. 在这里，会调用一个至关重要的函数，就是read()
> 5. 跳转到这个函数里面，read就是我们读取数据的关键函数

嗯。。这个函数还是蛮复杂的，先做下一个实验吧

---

## 5. Xargs

我没用过xargs，最开始可以说是一头雾水，包括最开始做的时候，甚至还不知道可以传递多行参数，改了半天。

整体思路就是先将当前右侧的参数读取，然后循环从标准输入中读取数据，遇到换行符，则执行命令，然后重置当前的参数和缓冲区为初始状态。

```c
#include "kernel/types.h"
#include "user/user.h"
#include "kernel/param.h"

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(2, "Usage: xargs command [args...]\n");
        exit(1);
    }
    char *cmd = argv[1];
    char *args[MAXARG];
    int i, n = 0;

    // 复制参数
    for (i = 1; i < argc && n < MAXARG - 1; i++) {
        args[n++] = argv[i];
    }
    int end = n;
    //方便重置索引
    char buf[512];
    int m = 0;
    while (read(0, &buf[m], 1) == 1) {
        if (buf[m] == '\n') {
            buf[m] = 0;
            args[n++] = &buf[0];

            // 参数必须以 NULL 结尾
            args[n] = 0;

            int fd = fork();
            if (fd == 0) {
                exec(cmd, args);
                fprintf(2, "xargs: exec failed\n");
                exit(1);
            }
            wait(0);

            // 索引重置
            m = 0;
            n = end;
        } else {
            m++;
        }
    }
    exit(0);
}
```

---

lab1给我感觉倒是没有太多关于os的知识，更多的是需要你去熟悉这个xv6的大体是什么样的，给他添加一些组件。

目前来看，收获更多的还得是xv6的教科书，那里面确实能够学到很多东西，给我的感觉就是比其他我看过的任何操作系统课对小白都要友好得多。
