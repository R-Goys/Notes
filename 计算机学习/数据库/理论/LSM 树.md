参考资料：
https://zhuanlan.zhihu.com/p/181498475
https://github.com/Vanilla-Beauty/tiny-lsm-go

LSM 树本身并不是一种严格的树状结构，而是一种存储结构，目前著名的 rocksdb、Leveldb 都是使用的这种存储结构。他通过顺序写来提高写入性能，其文件和内存的分层设计会降低一定的读性能，由于他的高性能写入，所以他相对比较流行。但是 mysql 不就是用了顺序写和分层设计吗？但是它并不是 lsm 树，只能说有异曲同工之妙，真正的存储结构并不是一样的。

![[Pasted image 20260109211638.png]]（图片摘自知乎）

## LSM 基本结构

首先要说的是 lsm 的分层结构，结构变换是 `memtable`（内存）-> `immutable memtable`（内存 -> `sstable`（磁盘）

`memtable` 是一个内存结构，通常会依靠 WAL 来保证数据可靠性，他用于保存最近的数据，会通过一定的结构组织这些数据。

`immutable memtable` 是在 `memtable` 到达一定大小，也就是 full 之后会转换为 `immutable memtable`，他是 `memtable` 转换为 `sstable` 的中间状态，在 `memtable` full 之后，他除了转换还会新创建一个 `memtable`，用来接管新的写操作，此时就不会因为 `immutable memtable` 转换而阻塞数据更新操作。

`sstable` 则是 lsm 在磁盘中的数据结构，可以通过索引和布隆过滤器来实现快速查找。通过上面的表述我们可以知道，`memtable` 是到达一定的大小就会转存为 `sstable` 到磁盘上，这也意味着每个 `sstable` 可能会存储相同的 key 记录，当然这样写性能会非常高，但是读取性能就会比较差，并且占用了多余的存储空间。

为了解决这个问题，lsm 会进行一定的 compact 操作。

## compact 策略

lsm 的 compact 操作有几种不同的策略，主要是在读写性能和空间占比上做的不同程度的权衡的策略。

### size-tiered

size-tiered 策略是一种滚雪球的模式，他将不同大小的 `sstable` 分为几个层级，当某一个层级的 `sstable` 达到 N，就会将他们合并，并根据 `sstable` 大小写入新的层级，然后旧的 `sstable` 就可以安全地删除了。

他的缺点就是**读放大**很明显，因为这个策略允许不同大小，数据范围可能重叠的 `sstable` 共存，我们寻找一个 key 的时候无法确定他的位置；还有一个缺点就是**空间放大**，合并时，会需要临时的空间存放，不仅如此，`sstable` 被合并的时机可能并不及时，导致磁盘使用率很低。当然他也有他的优点，就是写入性能非常优秀，因为他除了最开始需要一次写入，后续的磁盘 compact 迁移数据只会在同尺寸 `sstable` 到达一定数量时才会合并，合并次数是有限的。

### leveled

leveled 也是一种层级结构，和 size-tiered 的区别是，leveled 是有序层级，