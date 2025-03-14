## Redis

----

## 1. 基础语法

#### 1.1.数据类型

1. **String**: 最基本的类型，可以存储任何数据，例如文本或数字。示例值为 `hello world`。
2. **Hash**: 用于存储键值对，适合存储对象或结构体。示例值为 `{"name": "Jack", "age": 21}`。
3. **List**: 有序的字符串列表，适用于队列等场景。示例值为 `[A -> B -> C]`。
4. **Set**: 不重复的元素集合，适用于需要唯一性的场景。示例值为 `{A, B, C}`。
5. **SortedSet**: 有序且唯一的元素集合，每个元素有一个对应的分数，用于排序。示例值为 `{A: 1, B: 2, C: 3}`。
6. **GEO**: 用于处理地理数据，比如位置的经纬度。示例值为 `{A: (120.3, 30.5)}`。
7. **Bitmap**: 用于存储位图，可以支持高效的位运算。示例值为 `011011010110101011`。
8. **HyperLog**: 一种用于基数估算的数据结构，节省空间。示例值为 `011011010110101011`。

----

#### 1.2.通用命令

> 1. **`help @数据类型`**：得到相应数据类型会用到的命令.
> 2. **`KEYS ***???`**：获取符合模糊匹配的所有关键字，"?"代表一个字符，"*"代表任意个字符。
> 3. **`del [keyname] [keyname]....`**：删除key。
> 4. **`EXISTS [keyname]`**：查询key是否存在。
> 5. **`EXPIRE [keyname]`**：设置过期时间。
> 6. **`TTL [keyname]`**：查看过期时间，-1表示永久有效，-2表示不存在这个key。

----

#### 1.3.String类型

> 1. **`SET [key] [value]`**: 添加或修改一个已有的String类型的键值对。
> 2. **`GET [key]`**: 根据key获取String类型的value。
> 3. **`MSET [key1 value1] [key2 value2] ...`**: 批量添加多个String类型的键值对。
> 4. **`MGET [key1] [key2] ...`**: 根据多个key获取多个String类型的value。
> 5. **`INCR [key]`**: 让一个整型的key自增1。
> 6. **`INCRBY [key] [increment]`**: 让一个整型的key自增并指定步长，例如：`INCRBY num 2`让num值自增2。
> 7. **`INCRBYFLOAT [key] [increment]`**: 让一个浮点类型的数字自增并指定步长。
> 8. **`SETNX [key] [value]`或者`SET [key] [value] NX`**: 添加一个String类型的键值对，前提是这个key不存在，否则不执行。
> 9. **`SETEX [key] [seconds] [value]`或者`SET [key] EX [value]`**: 添加一个String类型的键值对，并且指定有效期（单位：秒）。

----

#### 1.4.key的层级结构

由于Redis中没有表这一结构，于是我们会需要key按照`项目名:业务名:类型:主键id`的方式命名，但并不固定，比如mysql里面的shopping库中的goods表的id为1的数据的key可以表示为`shopping:goods:1`，而这一个key对应的value可以是结构体(对象)序列化后的json字符串,这里值得一提的是，如果你用的`RDM`的redis图形化界面，这样的命名在图形化界面里面会以`树`的形式出现，显示很清晰，但是`Datagrip`这类软件貌似并不支持这个功能。

#### 1.5.Hash类型

```sql
key
├── field1: value1
├── field2: value2
└── field3: value3
...
```

哈希类型，它的value是一个无序字典，可以理解为key里面又存储了多个key的键值对，相较于上面json字符串形式存储数据有着一定的优势，那就是对json字符串中的单个数据进行修改很不方便，而hash类型则可以对单个字段进行CRUD。

**常用命令**:

> 1. **`HSET [key] [key] [field1] [value1] [field2] [value2]...`**: 添加或修改 hash 类型 key 的 field 的值。注：hmset也行，不过已经弃用了.
> 2. **`HGET [key] [field]`**: 获取一个 hash 类型 key 的 field 的值。
> 3. **`HMGET [key] [field1] [field2] ...`**: 批量获取多个 hash 类型 key 的 field 的值。
> 4. **`HGETALL [key]`**: 获取一个 hash 类型的 key 中的所有 field 和 value。
> 5. **`HKEYS [key]`**: 获取一个 hash 类型的 key 中的所有 field。
> 6. **`HVALS [key]`**: 获取一个 hash 类型的 key 中的所有 value。
> 7. **`HINCRBY [key] [field] [increment]`**: 让一个 hash 类型 key 的指定 field 值增加并指定步长。
> 8. **`HSETNX [key] [field] [value]`**: 添加一个 hash 类型 key 的 field 的值，前提是这个 field 不存在，否则不执行。

----

#### 1.6.List类型

可以看作是一个双向队列结构

**特征**：

- 有序
- 元素可以重复
- 支持插入和删除操作
- 查询速度一般

**常用命令**

> 1. **`LPUSH [key] [element] ...`**: 向列表左侧插入一个或多个元素。
> 2. **`LPOP [key]`**: 移除并返回列表左侧的第一个元素，没有则返回 nil。
> 3. **`RPUSH [key] [element] ...`**: 向列表右侧插入一个或多个元素。
> 4. **`RPOP [key]`**: 移除并返回列表右侧的第一个元素。
> 5. **`LRANGE [key] [start] [end]`**: 返回一段角标范围内的所有元素。
> 6. **`BLPOP [key] [timeout]`**: 与 `LPOP` 类似，在没有元素时等待指定时间。
> 7. **`BRPOP [key] [timeout]`**: 与 `RPOP` 类似，在没有元素时等待指定时间。

----

#### 1.7.Set类型

 相当于C++的 `unordered_set` 或者Java的`HashSet`，可以用于查看共同好友等。

**特征**：

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集、差集等功能

**常用命令**

> 1. **`SADD [key] [member] ...`**: 向 set 中添加一个或多个元素。
> 2. **`SREM [key] [member] ...`**: 移除 set 中的指定元素。
> 3. **`SCARD [key]`**: 返回 set 中元素的个数。
> 4. **`SISMEMBER [key] [member]`**: 判断一个元素是否存在于 set 中。
> 5. **`SMEMBERS [key]`**: 获取 set 中的所有元素。
> 6. **`SINTER [key1] [key2] ...`**: 求 key1 与 key2 的交集。
> 7. **`SDIFF [key1] [key2] ...`**: 求 key1 与 key2 的差集。
> 8. **`SUNION [key1] [key2] ...`**: 求 key1 和 key2 的并集。

---

#### 1.8.SortedSet类型

可以理解为C++中的Map,可以用于排行榜系统

**特征**：

- 可排序
- 元素不重复
- 查询速度快

**常用命令**

> - **`ZADD [key] [score] [member]`**: 添加一个或多个元素到 sorted set，如果已存在则更新其 score 值。
> - **`ZREM [key] [member]`**: 删除 sorted set 中的指定元素。
> - **`ZSCORE [key] [member]`**: 获取 sorted set 中指定元素的 score 值。
> - **`ZRANK [key] [member]`**: 获取 sorted set 中指定元素的排名。
> - **`ZCARD [key]`**: 获取 sorted set 中的元素个数。
> - **`ZCOUNT [key] [min] [max]`**: 统计 score 值在指定范围内的所有元素的个数。
> - **`ZINCRBY [key] [increment] [member]`**: 让 sorted set 中的指定元素自增，步长为指定的 increment 值。
> - **`ZRANGE [key] [min] [max]`**: 按照 score 升序排序，获取指定排名范围内的元素，在这里，查询的排名的范围为(min, max]
> - **`ZREVRANGE [key] [min] [max]`**: 按照 score 降序排序。
> - **`ZRANGEBYSCORE [key] [min] [max]`**: 按照 score 排序，获取指定 score 范围内的元素。
> - **`ZDIFF`、`ZINTER`、`ZUNION`**: 求差集、交集、并集。



----

## 2. 设计原理

redis底层使用C语言实现，这里主要分析底层数据结构

#### 2.1 **动态字符串(SDS)**

由于C底层的字符串数组一旦遇到'\0'就会认为这个字符串数组已经结束，意味着无法存储二进制数据（如图片、音频等），为此，redis底层维护了一种数据结构:

```c
struct __attribute__ ((__packed__)) sdshdr8 {
    
    uint8_t len;       // 当前字符串长度（不包含终止符 '\0'）
    
    uint8_t alloc;     // 分配的总容量（不包含 '\0'）
    
    unsigned char flags; // 低 3 位用于标识 SDS 类型（8/16/32/64）
    
    char buf[];        // 字符串数据（最后包含 '\0'）
};

```

这样，SDS便能自己控制字符串数组的结束，事实上，为了兼容C语言，在buf中的最后也会有'\0'。

另外，除了8bit位的sds，向上还有16/32/64bit位类型的sds数据结构，用于保存不同长度的字符串。

除此之外，SDS具有动态扩容的能力，在追加字符串的时候，会分配新的内存空间，并且和go中的切片类似，在字符串很小的时候，扩容的比例很大，字符串更大的时候，扩容的比例就会逐渐变小，这就是**内存预分配**。

**优势**：

1. 获取字符串长度时间复杂度为O(1)
2. 支持动态扩容，同时允许缩短字符串，但不会主动释放空间，而是等待未来的追加操作。
3. 由于分配的内存空间会在一定程度上大于使用的空间，所以会减少内存分配次数，提高性能
4. 二进制安全

----

#### 2.2 **IntSet**

字面意思，就是整数的Set集合，基于c语言中的整数数组实现，有序(二分查找实现)，支持动态扩容，数据结构如下：

```c
typedef struct intset {
    
    uint32_t encoding;	//编码方式，支持16/32/64位整数
    
    uint32_t length;	//元素个数
    
    int8_t contents[];	//保存集合的数据
    
} intset;
```

这里值得注意的是，数据的类型并不是由contents的类型`int8_t`决定的，而是由`encoding`字段决定的.

> 虽然`intset`结构将`contents`属性声明为`int8_t`类型的数组，但实际上`contents`数组并不保存任何`int8_t`类型的值，`contents`数组的真正类型取决于`encoding`属性的。

由于这里没有遵循C语言的规范，所以之后的关于IntSet的增删改查都是由redis自己实现的。

IntSet还支持**动态升级**，当新插入的数字超出了当前数字编码所能表示的范围，当前集合的所有数据都会被升级，比如说，当前集合存储的所有数据都是int16，而当前插入一个114514，超出了这个集合能表示的范围，此时，集合中的所有数据的数据类型都会升级为int32，并更新编码方式为32位编码，具体过程如下：

1. 更新编码方式，扩容。
2. 倒序将元素重新插入数组(正序插入会导致数据覆盖)。
3. 将待添加的元素插入在数组中合适的位置

----

#### 2.3 **Dict(字典)**

redis中的K-V键值对的映射关系，正是Dict这一数据结构实现的，也正相当于我们所熟知的哈希表。

一个节点的数据结构：

```c
struct dictEntry {
    void *key;  // 存储键（Key）

    union {  // 存储值（Value），支持不同类型
        void *val;      // 通用指针，可存储字符串、结构体等
        uint64_t u64;   // 存储无符号整数
        int64_t s64;    // 存储有符号整数
        double d;       // 存储双精度浮点数
    } v;

    struct dictEntry *next;  // 指向下一个 `dictEntry`，用于解决哈希冲突（拉链法法）
};
```

一个节点很好懂，并且可以看出，redis底层的哈希表是通过拉链法来实现的，当产生哈希冲突的时候，采取的是头插法，效率更高。

在最新的版本中，dictht被合并到了dict结构体中，数据结构如下：

```c
struct dict {
    dictType *type;           //哈希函数

    dictEntry **ht_table[2];  // 两个哈希表指针（ht_table[0] 正常使用，ht_table[1] 进行 rehash）
    unsigned long ht_used[2]; // 记录两个哈希表中存储的 entry 数量

    long rehashidx;           // 记录 rehash 进度，-1 表示没有进行 rehash

    //为了减少结构体的填充
    unsigned pauserehash : 15;  // 是否暂停 rehash
    unsigned useStoredKeyApi : 1; // 是否使用存储的 key API（内部优化）

    signed char ht_size_exp[2]; // 存储哈希表大小的指数（size = 1 << exp）
    int16_t pauseAutoResize;   // 是否暂停自动调整大小
    void *metadata[];          // 额外的元数据（可扩展）
};
```

**为什么这里会有两个哈希表指针？**

和go语言类似，这里涉及到我们的扩容操作，redis中也有着**负载因子**(存储元素和桶的比值)的概念，每次插入新的kv的时候，都会检查负载因子，同时在以下情况会触发rehash：

1. 当哈希表的负载因子 > 1.0，并且此时没有执行后台进程的时候
2. 负载因子 > 5.0的时候，会强制扩容

除了扩容，当然也有缩容的操作，当负载因子小于1/8，允许缩容(可能会受到其他条件的限制，而导致无法缩容)，当负载因子小于1/32时，会强制缩容。

两个方法的终点都是`int _dictResize(dict *d, unsigned long size, int* malloc_failed)`这个方法，在这里，会重新分配给`h_table[1]`新的空间，而`h_table[0]`作为我们旧的哈希表继续存在，一般来说，扩容时当前已经使用空间**used + 1**距离最近的一个2的n次方，而缩容则是**used**距离最近的2的n次方，此时，**rehash**才要刚刚开始。

##### **Rehash**

Rehash这一步的时候，扩容已经完成，此时首先为h_table[0]中的元素重新计算哈希值，并迁移到h_table[1]中，随后将h_table[0]的指针指向h_table[1]指向的空间，随后将h_table[1]指针置为NULL，此时我们的rehash就算完成了。

**重点来了**，事实上，重新分配内存之后，并没有直接进行数据迁移，而是进行**渐进式rehash**，每次在进行增删查改的时候，只迁移一个桶中的数据，所以并不会带给cpu过大的压力，因为如果一次性迁移完成的话，会导致cpu阻塞而无法执行其他操作，对用户十分不友好。

同时，在执行增删查改的时候，增只在新表中执行；删则是先从旧表中找，如果找到了直接删除，没找到再从新表中找；查询操作则是先查旧表，如果没找到，则查新表；修改操作则是先在旧表中找数据，找到则修改，没找到就在新表中找。



----

#### 2.4 **ZipList**

被称作**双端链表**，可以在头部和尾部实现O(1)的插入和删除，根据这个性质我感觉更像双端队列

ziplist是由一段特殊编码的**连续**内存块组成的，结构如下：

> zlbytes(存储链表占用的内存长度) -> zltail(记录头节点到尾节点的距离) -> zllen(节点数量) -> entry(节点) -> entry -> entry... -> zlend(结束标识)

其中，每个节点(entry)的长度并不固定，虽然源码中的结构体并不是真正的编码方式，还是在这里列一下吧：

```c
/* 该结构体用于解析 ziplist 中的条目元信息（非实际存储格式，仅为操作方便而设计） */
typedef struct zlentry {
    unsigned int prevrawlensize; // 存储【前一个节点长度】所需的字节数（1或5字节）
                                 // 规则：若前节点长度<254则用1字节，否则首字节0xFE+4字节长度
    
    unsigned int prevrawlen;     // 前一个节点的实际长度（字节数）
                                 // 作用：通过【当前地址 - prevrawlen】可快速定位前节点
    
    unsigned int lensize;        // 存储【当前节点类型/长度】所需的字节数
                                 // 字符串：1/2/5字节头 | 整数：固定1字节（类型和长度合并编码）
    
    unsigned int len;            // 当前节点数据的实际长度（字节数）
                                 // 字符串：字符数量（如"abc"为3）| 整数：根据类型占1/2/3/4/8字节
    
    unsigned int headersize;     // 节点头总大小 = prevrawlensize + lensize
                                 // 关键用途：p + headersize 直接跳转到数据区域
    
    unsigned char encoding;      // 编码类型标记（ZIP_STR_* 或 ZIP_INT_*）
                                 // 注意：0~12的4位小整数直接存于encoding，需特殊处理
    
    unsigned char *p;            // 指向当前节点起始地址（即prevrawlensize字段的位置）
                                 // 用途：修改节点时可直接操作原始内存
} zlentry;
```

看起来很复杂，我们可以简化为以下结构：

> previous_entry_length -> encoding(字符串/整数/长度) -> content

以上的数据在实际的存储结构中都可以灵活改变，也就是说**entry**长度并不固定，这节省了很大的内存

我们可以看到，每个节点事实上并没有存储前一个节点的指针和后一个节点的指针，而是记录了前一段节点和当前节点的长度来实现了一个**链表**，可以满足逆序和正序遍历，为什么不用指针呢？因为指针更占空间，我们知道，redis是内存存储数据，所以内存很宝贵！

值得一提的是，当存储类型为整数，且在一定范围内的时候，会直接将数值存储在encoding中！

##### **连锁更新问题**

**连锁更新**问题是什么？比方说，我们有很多长度为250\~253的entry节点，此时，往前面插入一个254长度的节点，那么此时previous_entry_length需要记录的长度>253，所以需要升级为5个字节来表示前一个节点的长度，那么此时，这个节点的总长度也会变，也就是说，之后的长度在250\~253之间的节点都会发送连锁的改变，进而导致插入更新一个节点会导致整个ziplist的变化，每次变化都会涉及到内存的申请，迁移，销毁，还有可能会牵扯到内核态的切换，进而带来性能开销。

虽然ziplist这个问题并没有被官方解决掉，但是却引入了一个新的数据结构**listpack**来解决这个问题，同时，这个问题发生的概率也是极低的，但是不代表不会发生！



----

#### 2.5 **QuickList**

是一个真正的**双端链表**，每一个节点都是一个**ZipList**

**为什么要这样做？**

ziplist虽然节省内存，但是如果内存占用过多，申请内存的效率就会很低，我们需要限制ziplist的大小，而怎么来存储大量的数据呢？答案就是使用多个ziplist，将他进行分片存储数据，然后由QuickList统一管理，同时，通过`list-max-ziplist-size`这个配置项来管理ziplist的最大大小，同时还会有一个压缩的配置选项`list-compress-depth`来控制压缩，压缩算法采取LZF，压缩可以减少内存占用，提高存储效率



---

#### 2.6 **SkipList(跳表)**

跳表首先是链表，以上两种表，虽然在头尾读写的时候性能优异，但是如果要在中间随机查找，性能就会大打折扣了，因此，还有一种数据结构叫做**跳表**，他有以下特点：

1. 元素升序排列
2. 每个节点可能包含多个指针，且跨度不同(跨越多个节点)，最多32层指针。

由于是升序排列，所以性能比较高，再加上**多个指针**的优势，查找的性能就更高了，这里加个图，来源于黑马程序员的redis教程的ppt，这个结构看起来其实很像B+树，而上方的指针则相当于索引，插入和查找的效率均为O(logN)![pic](.\assets\QQ_1741924759008.png)

另外的，skiplist的底层存储的数据就是**SDS**，而排序则是利用的索引，有一个字段score来帮助排序，性能和红黑树差不多，但是实现更为简单。(这很明显)



----

#### 2.7 **RedisObject**

redis中任意数据类型的键和值都会被封装非RedisObject，也叫做redis对象，数据结构如下：

```c
struct redisObject {
    unsigned type:4;      // 4 位表示对象的类型，zset，string，hash等类型
    unsigned encoding:4;  // 4 位表示对象的编码方式
    unsigned lru:LRU_BITS; // 最近一次被访问的时间
    int refcount;         // 引用计数，为0时，可以被回收
    void *ptr;            // 指向底层数据的指针
};
```

同时如果string类型被频繁的使用，那么每一个string都会有一个redisObject头来表示string，这就造成了更多的空间浪费，所以如果存储大量数据的时候，最好还是采取集合的形式。

同时，redis中的基本数据类型都是由这些底层设计的数据结构衍生而来：

| **Redis 数据类型**   | **小数据量的底层结构**    | **大数据量的底层结构**                         |
| -------------------- | ------------------------- | ---------------------------------------------- |
| **String**（字符串） | **SDS（简单动态字符串）** | **SDS（同样是 SDS，只是更大）**                |
| **List**（列表）     | **listpack（列表包）**    | **quicklist（快速列表，listpack + 双端链表）** |
| **Hash**（哈希）     | **listpack（列表包）**    | **dict（哈希表）**                             |
| **Set**（集合）      | **intset（整数集合）**    | **dict（哈希表，值为 NULL）**                  |
| **Zset**（有序集合） | **listpack（列表包）**    | **skiplist（跳表）+ dict（哈希表）**           |

对于redis7.0之前的版本，那么listpack替换为Ziplist即可。

#### --------------

#### 2.8 **String**

基本编码方式为Raw，存储上限为512MB，如果存储的SDS长度小于44字节，则会采用EMBSTR编码，此时object head和SDS是一段连续的空间，申请内存只需要调用一次内存分配函数，具有更高的效率，所以使用string时尽量不要超过这个值。

如果存储的字符串是整数，并且在范围内，则会采取INT编码，直接保存在RedisObject head的ptr指针上面，这样更加节省了内存。

#### 2.9 **List**

Redis中的list支持快速操作首尾的元素。

因此，list由Quicklist和ZipList(最新版为listPack)实现

当数据量很小的时候，会采用ListPack作为底层数据结构，而在数据逐渐增多的时候，则会升级为QuickList，在源代码中，list底层结构的升级和降级具体体现在`static void listTypeTryConversionRaw(参数)`，会根据当前list的状态，判断是否需要升级或者降级操作。

----

#### 2.10 **Set**

Set，也就是集合，具有无序性，唯一性(需要判断元素是否存在)，可以求交集并集和差集。(梦回高中数学)

综上，Set对**查询**的要求比较高，于是底层采用了Dict(字典)编码，Key来存储数值，而Value统一为NULL，与此同时，当存储的数据都为整数的时候，如果元素数量不超过一定值，统一采取IntSet编码，这个定值可以通过修改`set-max-intset-entries`来修改。

#### 2.11 **ZSet**

有序的集合，每个元素都需要设置一个member和score值，member就相当于是Set中的元素，具有唯一性，顺序通过score来确定。

 有序，同时还需要member->score的映射，具有快速查找的功能，Skiplist可以存储member和score，实现有序性，而Dict可以存储member到score的映射，实现快速查找，因此，ZSet底层采用的是SkipList和Dict结合的形式来实现这样的**有序集合**，

