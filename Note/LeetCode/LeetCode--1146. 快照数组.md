[1146. 快照数组](https://leetcode.cn/problems/snapshot-array/)

> 实现支持下列接口的「快照数组」- SnapshotArray：
>
> - `SnapshotArray(int length)` - 初始化一个与指定长度相等的 类数组 的数据结构。**初始时，每个元素都等于** **0**。
> - `void set(index, val)` - 会将指定索引 `index` 处的元素设置为 `val`。
> - `int snap()` - 获取该数组的快照，并返回快照的编号 `snap_id`（快照号是调用 `snap()` 的总次数减去 `1`）。
> - `int get(index, snap_id)` - 根据指定的 `snap_id` 选择快照，并返回该快照指定索引 `index` 的值。

---

这思路真难想出来，将每个索引设置为下标，当需要设置历史值的时候，就则向后追加元素，当需要 Get 的时候，就需要使用二分查找，先获取指定下标对应的一串历史值，然后二分查找对应的快照 id 即可。

```go
type SnapshotArray struct {
    curSnapId int
    history   [][][2]int
}


func Constructor(length int) SnapshotArray {
    return SnapshotArray{ 
        history: make([][][2]int, length),
    }
}


func (this *SnapshotArray) Set(index int, val int)  {
    this.history[index] = append(this.history[index], [2]int{this.curSnapId, val})
}


func (this *SnapshotArray) Snap() int {
    this.curSnapId ++
    return this.curSnapId - 1
}


func (this *SnapshotArray) Get(index int, snap_id int) int {
    h := this.history[index]

    idx := sort.Search(len(h), func(j int) bool { return h[j][0] >= snap_id + 1 }) - 1
    if idx >= 0 {
        return h[idx][1]
    }
    return 0
}

```

