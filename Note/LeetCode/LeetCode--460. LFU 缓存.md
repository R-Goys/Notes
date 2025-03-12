[460. LFU 缓存](https://leetcode.cn/problems/lfu-cache/)

> 请你为 [最不经常使用（LFU）](https://baike.baidu.com/item/缓存算法)缓存算法设计并实现数据结构。
>
> 实现 `LFUCache` 类：
>
> - `LFUCache(int capacity)` - 用数据结构的容量 `capacity` 初始化对象
> - `int get(int key)` - 如果键 `key` 存在于缓存中，则获取键的值，否则返回 `-1` 。
> - `void put(int key, int value)` - 如果键 `key` 已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量 `capacity` 时，则应该在插入新项之前，移除最不经常使用的项。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除 **最久未使用** 的键。
>
> 为了确定最不常使用的键，可以为缓存中的每个键维护一个 **使用计数器** 。使用计数最小的键是最久未使用的键。
>
> 当一个键首次插入到缓存中时，它的使用计数器被设置为 `1` (由于 put 操作)。对缓存中的键执行 `get` 或 `put` 操作，使用计数器的值将会递增。
>
> 函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

---

真难写啊我去

思路如下：

将使用次数相同的KV放在同一个链表中维护，与此同时，所有的kv都需要存入哈希表来实现快速查找

这样在执行`PUT`操作的时候，会先看哈希表中有无对应的键，随后看是否存在对应的k，如果有，就会添加一个新的节点，并且在超出容量时，能够在当前使用次数最低的链表中删除一个即可，随后将kv推入到使用次数为1的链表中，如果没有，创建即可。

当然，如果存在对应的键，就会根据这个k，去查找相对应的元素，里面存入了使用次数和kv，随后更新value，删除在原链表中的节点，推入新链表即可。

执行`GET`操作时，向上面一样，更新一下当前kv的使用次数，更新这个节点在链表中的位置，直接返回v即可。

```go
type LFUCache struct {
      capacity int
      minFreq int
      KeyToNode map[int]*list.Element
      FreqToList map[int]*list.List
}

type entry struct {
    key, value, freq int
}


func Constructor(capacity int) LFUCache {
    return LFUCache{
        capacity: capacity,
        KeyToNode: map[int]*list.Element{},
        FreqToList: map[int]*list.List{},
    }
}

func (this *LFUCache) getEntry(key int) *entry {
    node := this.KeyToNode[key]
    if node == nil {
        return nil
    }
    e := node.Value.(*entry)
    lst := this.FreqToList[e.freq]
    lst.Remove(node)
    if lst.Len() == 0 {
        delete(this.FreqToList, e.freq)
        if this.minFreq == e.freq {
            this.minFreq ++
        }
    }
    e.freq ++ 
    this.pushFront(e)
    return e
}


func (this *LFUCache) Get(key int) int {
    if e := this.getEntry(key); e != nil {
        return e.value
    }
    return -1
}


func (this *LFUCache) Put(key int, value int)  {
    if e := this.getEntry(key); e != nil {
        e.value = value
        return
    }
    if len(this.KeyToNode) == this.capacity {
        lst := this.FreqToList[this.minFreq]
        delete(this.KeyToNode, lst.Remove(lst.Back()).(*entry).key)
        if lst.Len() == 0 {
            delete(this.FreqToList, this.minFreq)
        }
    }
    this.pushFront(&entry{key, value, 1})
    this.minFreq = 1
}

func (this *LFUCache) pushFront(e *entry) {
    if _, ok := this.FreqToList[e.freq]; !ok {
        this.FreqToList[e.freq] = list.New()
    }
    this.KeyToNode[e.key] = this.FreqToList[e.freq].PushFront(e)
}
```

