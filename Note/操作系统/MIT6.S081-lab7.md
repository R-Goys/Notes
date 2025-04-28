# MIT6.S081-lab7

## 1. Memory allocator

这个实验就是让我们去将所有的页分配到每个运行的 CPU 里面，在每个 cpu 分配页的时候，会首先从自己维护的链表中取出空闲页，如果发现不足，就会从其他的 cpu 中窃取页，以此将页表分散到每个 cpu 来减少锁导致的开销。

如何实现？根据 hint ，我们可以了解到 NCPU 这个常量，通过它，我们可以初始化一个长度为 NCPU 的 kmem 数组，并且需要为他们分别分配锁，由于需要用到一些关于 cpu 的函数，我们需要引入 "proc.h" 这个头文件

```c
struct run {
  struct run *next;
};

struct {
  struct spinlock lock;
  struct run *freelist;
} kmem[NCPU];

char* kmem_lock_name[NCPU] = {
  "kmem1", "kmem2", "kmem3", "kmem4", "kmem5", "kmem6", "kmem7", "kmem8"
};
```

不需要将 kmem 放在 cpu 结构体中，会徒增麻烦。

接下来，我们会发现有很多地方都爆红了，但是别担心，首先需要更新我们的 kinit：

```c
void
kinit()
{
  for(int i = 0; i < NCPU; i++) {
    // 这里发生改变，为每个 cpu 初始化一个锁。
    initlock(&kmem[i].lock, kmem_lock_name[i]);
  }
  freerange(end, (void*)PHYSTOP);
}
```

freerange 我们没什么好修改的，但是它调用了一个 kfree：

```c
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;
  // 获取当前 cpu 的 id
  int cpu = cpuid();
  // 分配页。
  acquire(&kmem[cpu].lock);
  r->next = kmem[cpu].freelist;
  kmem[cpu].freelist = r;
  release(&kmem[cpu].lock);
}
```

最后，我们需要给分配页表的函数 kalloc 进行大改！

```c
// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if out of memory.
void *
kalloc(void)
{
  struct run *r;

  int cpu = cpuid(); // 当前 CPU id
  acquire(&kmem[cpu].lock); // 锁住本 CPU 的内存分配器

  r = kmem[cpu].freelist; // 尝试从自己的空闲链表拿一页

  release(&kmem[cpu].lock);

  // 如果自己链表空了，去偷别人的页面
  if(!r) {
    int steal_pages = 0;
    for(int i = 0; i < NCPU; i++) {
      if (i == cpu) continue; // 不偷自己

      acquire(&kmem[i].lock); // 锁住第 i 个 CPU 的链表，在 while 前面锁是为了读取的数据是正确的
        						  // 但是之后需在正确的位置解锁
      while (kmem[i].freelist) {
        // 取出 i 的 freelist 中最前面的页面
        struct run* newpage = kmem[i].freelist;
        kmem[i].freelist = newpage->next;
        release(&kmem[i].lock);

        // 将偷来的页面加到自己链表头
        acquire(&kmem[cpu].lock);
        newpage->next = kmem[cpu].freelist;
        kmem[cpu].freelist = newpage;
        release(&kmem[cpu].lock);

        steal_pages++;

        // 如果偷够了 32 页，停止偷
        if(steal_pages == 32) {
          goto done;
        }

        // 可能会继续偷 i 的下一页，重新加锁
        acquire(&kmem[i].lock);
      }
      release(&kmem[i].lock); // i 没得偷了，解锁
    }

    // 如果一个页面都没偷到，说明系统真的内存用光了
    if(steal_pages == 0) {
      return 0;
    }
  }

done:
  // 完成，重新分配页表
  acquire(&kmem[cpu].lock);
  r = kmem[cpu].freelist;

  if(r)
    kmem[cpu].freelist = r->next;

  release(&kmem[cpu].lock);

  if(r)
    memset((char*)r, 5, PGSIZE);
  return (void*)r;
}
```

`make qemu` 之后输入 `kalloctest` ：

```bash
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem1: #test-and-set 0 #acquire() 35444
lock: kmem2: #test-and-set 0 #acquire() 298237
lock: kmem3: #test-and-set 0 #acquire() 299779
lock: bcache: #test-and-set 0 #acquire() 1256
--- top 5 contended locks:
lock: proc: #test-and-set 13895 #acquire() 401935
lock: uart: #test-and-set 8128 #acquire() 54
lock: virtio_disk: #test-and-set 5402 #acquire() 129
lock: proc: #test-and-set 3591 #acquire() 1228
lock: pr: #test-and-set 3207 #acquire() 5
tot= 0
test1 OK
start test2
total free number of pages: 32468 (out of 32768)
.....
test2 OK
start test3
..........child done 100000
--- lock kmem/bcache stats
lock: kmem1: #test-and-set 1316 #acquire() 443997
lock: kmem2: #test-and-set 2945 #acquire() 3199543
lock: kmem3: #test-and-set 5290 #acquire() 4120660
lock: kmem4: #test-and-set 0 #acquire() 122
lock: kmem5: #test-and-set 0 #acquire() 122
lock: kmem6: #test-and-set 0 #acquire() 122
lock: kmem7: #test-and-set 0 #acquire() 122
lock: kmem8: #test-and-set 0 #acquire() 122
lock: bcache: #test-and-set 0 #acquire() 1356
--- top 5 contended locks:
lock: uart: #test-and-set 66939 #acquire() 703
lock: proc: #test-and-set 20781 #acquire() 3320533
lock: proc: #test-and-set 14777 #acquire() 807080
lock: virtio_disk: #test-and-set 5402 #acquire() 129
lock: proc: #test-and-set 5389 #acquire() 6673
tot= 9551

test3 OK
```

完成。

## Buffer cache

实验的目的是降低 cache 锁的粒度，根据实验的 hint ，我们可以采取**哈希桶**，每个桶单独占有一把锁，一部分缓存 cache 链表，以此来提高并行性。

桶的数量可以很随机，我这里选取 13 ，因为总共的 cache 块也不多，并且我们需要构造一个哈希函数，以此来返回对应的哈希表中的位置：

```c
#define NBUCKET 13

#define HASH(dev, blockno) ((((unsigned int)(dev)) << 16 | (blockno)) % NBUCKET)

struct bucket {
  struct spinlock lock;
  struct buf *head;
};

struct {
  struct spinlock lock;
  struct buf buf[NBUF];

  // Linked list of all buffers, through prev/next.
  // Sorted by how recently the buffer was used.
  // head.next is most recent, head.prev is least.
  struct bucket buckets[NBUCKET];
} bcache;
```

随后，我们也会看到一堆爆红，我们首先需要做修改的就是 binit ，他是初始化我们的 bcache 链表的，我们可以把它改成均匀分布在每个哈希桶中，也可以单独挂在一个桶里面，这里为了方便，直接挂在一个桶里面了：

```c
void
binit(void)
{
  for (int i = 0; i < NBUCKET; i++) {
    initlock(&bcache.buckets[i].lock, "bcache.bucket");  // 初始化每个桶的锁
    bcache.buckets[i].head = 0;  // 初始化每个桶的链表头为空
  }
  initlock(&bcache.lock, "bcache");  // 初始化全局锁
  for(int i = 0; i < NBUF; i++) {
    struct buf *b = &bcache.buf[i];
    int bucket = 0;
    b->next = bcache.buckets[bucket].head;
    b->prev = 0;
    if (bcache.buckets[bucket].head != 0) {
      bcache.buckets[bucket].head->prev = b;
    }
    bcache.buckets[bucket].head = b;
    initsleeplock(&b->lock, "buffer");
  }
}
```

随后是我们的重点，也就是 bget ，可以说我们的核心逻辑就在这里面，最难的也是这部分的内容，和原来一样，我们先找到对应的桶，然后在对应的桶里面寻找对应的块缓存是否存在，如果不存在，那就去寻找我们的空闲 cache ，当然，如果不存在空闲 cache ，就从其他的桶中窃取，这里的控制很有意思，我在这里试了很多方法，发现都不能 pass 这个实验，最终，只能加一把大锁来保平安，防止死锁的情况。

```c
static struct buf*
bget(uint dev, uint blockno)
{
  struct buf *b;

  int bucket = HASH(dev, blockno);  // 计算桶号
  acquire(&bcache.buckets[bucket].lock);  // 给目标桶加锁

  // 查看这个块是否被缓存到目标桶
  for(b = bcache.buckets[bucket].head; b; b = b->next){
    if(b->dev == dev && b->blockno == blockno){
      b->refcnt++;
      release(&bcache.buckets[bucket].lock);  // 解锁目标桶
      acquiresleep(&b->lock);  // 给目标缓存块加锁
      return b;
    }
  }

  // 缓存未命中，在目标桶中找一块干净的块
  for(b = bcache.buckets[bucket].head; b; b = b->next){
    if(b->refcnt == 0) {
      b->dev = dev;
      b->blockno = blockno;
      b->valid = 0;
      b->refcnt ++;
      release(&bcache.buckets[bucket].lock);  // 解锁目标桶
      acquiresleep(&b->lock);  // 给目标缓存块加锁
      return b;
    }
  }
  release(&bcache.buckets[bucket].lock);  // 解锁目标桶
  acquire(&bcache.lock);
  for (int i = 0; i < NBUCKET; i++) {
    if (i == bucket) continue;
    acquire(&bcache.buckets[i].lock); // 获取当前桶的锁
    for (b = bcache.buckets[i].head; b; b = b->next) {
      if (b->refcnt == 0) {
        // 从当前桶移除块
        if (b->prev) {
          b->prev->next = b->next;
        } else {
          bcache.buckets[i].head = b->next;
        }
        if (b->next) {
          b->next->prev = b->prev;
        }
        // 将块添加到目标桶头部
        b->prev = 0;
        b->next = bcache.buckets[bucket].head;
        if (bcache.buckets[bucket].head) {
          bcache.buckets[bucket].head->prev = b;
        }
        bcache.buckets[bucket].head = b;

        b->dev = dev;
        b->blockno = blockno;
        b->valid = 0;
        b->refcnt = 1;

        release(&bcache.buckets[i].lock);
        release(&bcache.lock);
        acquiresleep(&b->lock);
        return b;
      }
    }
    release(&bcache.buckets[i].lock); // 释放当前桶的锁
  }
  release(&bcache.lock);
  panic("bget: no buffers");
}
```

随后关于 brelse 的改动也是很重要的，由于我们维护的哈希桶，查找快速，这里就直接采取引用数减一的模式。

```c
void
brelse(struct buf *b)
{
  int bucket = HASH(b->dev, b->blockno);
  if(!holdingsleep(&b->lock))
    panic("brelse");

  releasesleep(&b->lock);
  acquire(&bcache.buckets[bucket].lock);
  b->refcnt--;
  release(&bcache.buckets[bucket].lock);
}
```

最后，不那么重要的，直接修改一下几个函数的锁就行了：

```c
void
bpin(struct buf *b) {
  int bucket = HASH(b->dev, b->blockno);
  acquire(&bcache.buckets[bucket].lock);
  b->refcnt++;
  release(&bcache.buckets[bucket].lock);
}

void
bunpin(struct buf *b) {
  int bucket = HASH(b->dev, b->blockno);
  acquire(&bcache.buckets[bucket].lock);  
  if(b->refcnt > 0) b->refcnt--;
  release(&bcache.buckets[bucket].lock);
}
```

---

有点疲劳，写的比较马虎，明天再回来补充吧。