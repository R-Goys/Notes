参考资料：
https://zhuanlan.zhihu.com/p/181498475
https://github.com/Vanilla-Beauty/tiny-lsm-go

LSM 树本身并不是一种严格的树状结构，而是一种存储结构，目前著名的 rocksdb、Leveldb 都是使用的这种存储结构。他通过顺序写来提高写入性能，其文件和内存的分层设计会降低一定的读性能，由于他的高性能写入，所以他相对比较流行。但是 mysql 不就是用了顺序写和分层设计吗？但是它并不是 lsm 树，只能说有异曲同工之妙，真正的存储结构并不是一样的。

![[Pasted image 20260109211638.png]]（图片摘自知乎）

## LSM 基本结构

首先要说的是 lsm 的分层结构，结构变换是 memtable（内存）-> immutable memtable（内存 -> sstable（磁盘）

memtable 是一个内存结构，通常会依靠 WAL 来保证数据可靠性，他用于保存最近的数据，会通过一定的结构组织这些数据。

immutable memtable 是在 memtable 到达一定大小，也就是 full 之后会转换为 immutable memtable，他是 memtable 转换为 sstable 的中间状态，在 memtable full 之后，他除了转换还会新创建一个 memtable，用来接管新的写操作，此时就不会因为 immutable memtable 转换而阻塞数据更新操作。

sstable 则是 lsm 在磁盘中的数据结构，可以通过索引和布隆过滤器来实现快速查找。通过上面的表述我们可以知道，memtable 是到达一定的大小就会转存为 sstable 到磁盘上，这也意味着每个 sstable 可能会存储相同的 key 记录，当然这样写性能会非常高，但是读取性能就会比较差，并且占用了多余的存储空间。

为了解决这个问题，lsm 会进行一定的 compact 操作。

## compact 策略

