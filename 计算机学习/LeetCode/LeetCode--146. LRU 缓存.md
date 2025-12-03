[146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)

>请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。
>
>实现 `LRUCache` 类：
>
>- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
>- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
>- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。
>
>函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

----

## 前言

好题，想写一下，虽然之前用C++写过一次，但是不太熟悉c++的面向对象，也是云里雾里的，只会写个函数，这次也想熟悉一下golang的container/list。

## 正文

由于要实现LRU的算法，最主要的就是要实现如何存储最近最少使用的Key，最合适的就是链表，至于根据Key来寻找Value的值，自然就是哈希表了。

于是思路就是这样的：

1. Get()方法：根据给定的Key，在哈希表中查找，如果不存在，返回-1，如果存在则返回对应的val，并将我们维护的链表的对应的Key-Value更新到链表的最前面。
2. Put()方法：先查找哈希表中是否存储了这一Key，如果已经存储，则更新Value的值了；如果没有存储，则先确认缓存的数据是否超出了限制，如果没有超出，则直接加入哈希表即可，并将相应的Key-Value推入到链表的最前面；如果超出了，则找到链表的尾部，找到相应的key值，删除哈希表中的对应的Key-Value，并删除链表尾部元素，再将新的Key-Value加入哈希表和链表前方。

代码如下：

```go
type LRUCache struct {
	capacity int
	kv       map[int]*list.Element
	KeyList  *list.List
}

type KV struct {
	Key   int
	Value int
}

func Constructor(capacity int) LRUCache {
	return LRUCache{
		capacity: capacity,
		kv:       make(map[int]*list.Element),
		KeyList:  list.New(),
	}
}

func (this *LRUCache) Get(key int) int {
	if elem, ok := this.kv[key]; ok {
		this.KeyList.MoveToFront(elem)
		return elem.Value.(KV).Value
	}
	return -1
}

func (this *LRUCache) Put(key int, value int) {
	if node := this.kv[key]; node != nil {
		node.Value = KV{
			Key:   key,
			Value: value,
		}
		this.KeyList.MoveToFront(node)
		return
	}
	if this.KeyList.Len() >= this.capacity {
		oldest := this.KeyList.Back()
		this.KeyList.Remove(oldest)
		delete(this.kv, oldest.Value.(KV).Key)
	}
	KeyValue := KV{Key: key, Value: value}
	elem := this.KeyList.PushFront(KeyValue)
	this.kv[key] = elem
}
```

---

## 结语

Golang里面有这么一个list真是太棒了~