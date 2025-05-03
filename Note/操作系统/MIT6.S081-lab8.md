# MIT6.S081-lab8

## 1. Large files

从 lecture 我们可以知道，我们目前的单个文件的最大大小很小，这是因为我们能够索引的索引块范围很小，实际上，目前的索引只有直接索引和一级索引，而这个实验就是需要我们去实现二级索引，我们通过阅读 bmap 和 itrunc 可以发现，我们的一级索引实际上是通过分配一个数据块来进行索引了，所以我们二级索引也需要利用这个思路。

hint 没啥太重要的，这个 lab 很简单，但是依旧需要我们去根据 hint 做一些微操：

```c
struct inode {
  uint dev;           // Device number
  uint inum;          // Inode number
  int ref;            // Reference count
  struct sleeplock lock; // protects everything below here
  int valid;          // inode has been read from disk?

  short type;         // copy of disk inode
  short major;
  short minor;
  short nlink;
  uint size;
  // 此处修改为 + 2 ，增加索引位。
  uint addrs[NDIRECT+2];
};
```

fs.h

```c
// 这里根据 hint 需要修改为 11
#define NDIRECT 11
#define NINDIRECT (BSIZE / sizeof(uint))
// 修改最大文件的大小
#define MAXFILE (NDIRECT + NINDIRECT + NINDIRECT * NINDIRECT)

// On-disk inode structure
struct dinode {
  short type;           // File type
  short major;          // Major device number (T_DEVICE only)
  short minor;          // Minor device number (T_DEVICE only)
  short nlink;          // Number of links to inode in file system
  uint size;            // Size of file (bytes)
  // 修改，理由同 file.h
  uint addrs[NDIRECT+2];   // Data block addresses
};
```

然后，根据我们之前的前置知识，我们可以知道，我们文件的数据的地址都是通过 bmap 来索引的，而我们还可以通过 hint 来知道，我们需要对 itrunc 进行操作，所以剩下的就是直接对我们的 bmap 和 itrunc 进行操作了，我们的二级索引会先分配一个数据块，然后这个数据块的每一位又会分配一个数据块，最后一级的数据块才是真正存放地址的数据，这就叫二级索引，所以按着这个思路，这里纯爽局，直接跟着抄就行了，：

```c
static uint
bmap(struct inode *ip, uint bn)
{
  uint addr, *a;
  struct buf *bp;

  if(bn < NDIRECT){
    if((addr = ip->addrs[bn]) == 0){
      addr = balloc(ip->dev);
      if(addr == 0)
        return 0;
      ip->addrs[bn] = addr;
    }
    return addr;
  }
  bn -= NDIRECT;

  if(bn < NINDIRECT){
    // Load indirect block, allocating if necessary.
    if((addr = ip->addrs[NDIRECT]) == 0){
      addr = balloc(ip->dev);
      if(addr == 0)
        return 0;
      ip->addrs[NDIRECT] = addr;
    }
    bp = bread(ip->dev, addr);
    a = (uint*)bp->data;
    if((addr = a[bn]) == 0){
      addr = balloc(ip->dev);
      if(addr){
        a[bn] = addr;
        log_write(bp);
      }
    }
    brelse(bp);
    return addr;
  }
  
  bn -= NINDIRECT;

  if(bn < NINDIRECT * NINDIRECT) {
    // 分配数据块作为索引
    if((addr = ip->addrs[NDIRECT + 1]) == 0) {
      addr = balloc(ip->dev);
      if(addr == 0) {
        return 0;
      }
      ip->addrs[NDIRECT + 1] = addr; 
    }
    // 第一层索引
    bp = bread(ip->dev, addr);
    a = (uint*)bp->data;
    if((addr = a[bn / NINDIRECT]) == 0) {
      addr = balloc(ip->dev);
      if(addr) {
        a[bn / NINDIRECT] = addr;
        log_write(bp);
      }
    } 
    brelse(bp);
    // 第二层索引
    bn %= NINDIRECT;
    bp = bread(ip->dev, addr);
    a = (uint*)bp->data;
    if((addr = a[bn]) == 0) {
      addr = balloc(ip->dev);
      if(addr) {
        a[bn] = addr;
        log_write(bp);
      }
    }
    brelse(bp);
    return addr;
  }

  panic("bmap: out of range");
}

// Truncate inode (discard contents).
// Caller must hold ip->lock.
void
itrunc(struct inode *ip)
{
  // 这里加上 k，bp2 和 b
  int i, j, k;
  struct buf *bp, *bp2;
  uint *a, *b;

  for(i = 0; i < NDIRECT; i++){
    if(ip->addrs[i]){
      bfree(ip->dev, ip->addrs[i]);
      ip->addrs[i] = 0;
    }
  }

  if(ip->addrs[NDIRECT]){
    bp = bread(ip->dev, ip->addrs[NDIRECT]);
    a = (uint*)bp->data;
    for(j = 0; j < NINDIRECT; j++){
      if(a[j])
        bfree(ip->dev, a[j]);
    }
    brelse(bp);
    bfree(ip->dev, ip->addrs[NDIRECT]);
    ip->addrs[NDIRECT] = 0;
  }

  // 修改的地方
  if(ip->addrs[NDIRECT + 1]) {
    bp = bread(ip->dev, ip->addrs[NDIRECT + 1]);
    a = (uint*)bp->data;
    for(j = 0; j < NINDIRECT; j ++) {
      if(a[j]) {
        bp2 = bread(ip->dev, a[j]);
        b = (uint*)bp2->data;
        for(k = 0; k < NINDIRECT; k ++) {
          if(b[k]) {
            bfree(ip->dev, b[k]);
          }
        }
        brelse(bp2);
        bfree(ip->dev, a[j]);
      }
    }
    brelse(bp);
    bfree(ip->dev, ip->addrs[NDIRECT + 1]);
    ip->addrs[NDIRECT + 1] = 0;
  }
  ip->size = 0;
  iupdate(ip);
}
```

完成。

---

## 2. Symbolic links

这个实验也不是很难，我们的目的是实现一个 symlink 的系统调用，传入 target 和 path 的参数，其中，当我们使用了这个 symlink 系统调用之后，我们再次 open 其中的 path 的时候，会直接打开 target 对应的文件，这就是我们的链接。

由 hint 可知，我们先需要做一些准备工作：

kernel/stat.h

```c
#define T_DIR     1   // Directory
#define T_FILE    2   // File
#define T_DEVICE  3   // Device
// 添加的，这里随便写啥，只需要不重复
#define T_SYMLINK 4   // LAB 8
```

kernel/fcntl.h

```c
#define O_RDONLY  0x000
#define O_WRONLY  0x001
#define O_RDWR    0x002
#define O_CREATE  0x200
#define O_TRUNC   0x400
// 这里只要换成二进制不和上面重复就可以了
#define O_NOFOLLOW 0x004
```

如果你希望在 xv6 启动的时候调试：

makefile

```makefile
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
	#添加这里
	$U/_symlinktest\
```

usys.pl

```pascal
entry("symlink");
```

user.h

```c
int symlink(char *target, char *path);
```

syscall.h

```c
#define SYS_symlink 22
```

syscall.c

```c
extern uint64 sys_symlink(void);

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
// 添加这一行
[SYS_symlink] sys_symlink,
};
```

最后，我们可以在 sysfile.c 中编写我们的代码，首先，关于这个系统调用，由于它原本并不会存在，所以我们需要先创建这个文件，并且存储我们的 target ，如何存储？通过 hint 可以知道，我们可以将这个 target 存储到文件的数据块中：

```c
uint64 sys_symlink(void) {
  char target[MAXPATH], path[MAXPATH];
  struct inode *ip;
  argstr(0, target, MAXPATH);
  argstr(1, path, MAXPATH);

  begin_op();
  ip = create(path, T_SYMLINK, 0, 0);
  if(ip == 0) {
    end_op();
    return -1;
  }
  if (writei(ip, 0, (uint64)target, 0, strlen(target)) < 0) {
    end_op();
    return -1;
  }
  iunlockput(ip);

  end_op();
  return 0;
}
```

这里需要注意的就是事务的开启和关闭，通过之前的前置学习我们可以知道，通过 writei 可以将数据写入数据块中，这样我们就算是成功链接了。

最后，我们在 open 系统调用的时候，需要能够识别这个文件是一个链接文件，同时，需要保证能够递归查找一定的深度，这里我们可以直接仿照原本的 open 使用 namei 去获取对应的 fd ，然后通过 readi 获取链接的 target。
```c
uint64
sys_open(void)
{
  ...
  if(omode & O_CREATE){
    ip = create(path, T_FILE, 0, 0);
    if(ip == 0){
      end_op();
      return -1;
    }
  } else {
    if((ip = namei(path)) == 0){
      end_op();
      return -1;
    }
    ilock(ip);
    // 修改的部分
    int depth = 0;
    // 如果对应的 inode 是链接类型并且没有 NOFOLLOW 标志位
    while(ip->type == T_SYMLINK && (omode & O_NOFOLLOW) == 0) {
      // 如果递归过多层，及时退出
      if(++depth > 10) {
        iunlockput(ip);
        end_op();
        return -1;
      }
      // 注意，这里不清空数据，我们的测试样例过不了！
      memset(path, 0, MAXPATH);
      if(readi(ip, 0, (uint64)path, 0, MAXPATH) < 0) {
        iunlockput(ip);
        end_op();
        return -1;
      }
      iunlockput(ip);
      // 读取这个文件对应的 inode
      if((ip = namei(path)) == 0){
        end_op();
        return -1;
      }
      ilock(ip);
    }
    //此处跳出了循环，代表找到了最终的文件
    // 以上为修改的部分
    if(ip->type == T_DIR && omode != O_RDONLY){
      iunlockput(ip);
      end_op();
      return -1;
    }
  }
  ...

  return fd;
}
```

最后，我们可以通过所有的测试：

```bash
== Test running bigfile == 
$ make qemu-gdb
running bigfile: OK (139.0s) 
== Test running symlinktest == 
$ make qemu-gdb
(1.2s) 
== Test   symlinktest: symlinks == 
  symlinktest: symlinks: OK 
== Test   symlinktest: concurrent symlinks == 
  symlinktest: concurrent symlinks: OK 
== Test usertests == 
$ make qemu-gdb
usertests: OK (188.3s) 
```

