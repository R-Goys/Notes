[3508. 设计路由器](https://leetcode.cn/problems/implement-router/)

> 请你设计一个数据结构来高效管理网络路由器中的数据包。每个数据包包含以下属性：
>
> - `source`：生成该数据包的机器的唯一标识符。
> - `destination`：目标机器的唯一标识符。
> - `timestamp`：该数据包到达路由器的时间戳。
>
> 实现 `Router` 类：
>
> `Router(int memoryLimit)`：初始化路由器对象，并设置固定的内存限制。
>
> - `memoryLimit` 是路由器在任意时间点可以存储的 **最大** 数据包数量。
> - 如果添加一个新数据包会超过这个限制，则必须移除 **最旧的** 数据包以腾出空间。
>
> `bool addPacket(int source, int destination, int timestamp)`：将具有给定属性的数据包添加到路由器。
>
> - 如果路由器中已经存在一个具有相同 `source`、`destination` 和 `timestamp` 的数据包，则视为重复数据包。
> - 如果数据包成功添加（即不是重复数据包），返回 `true`；否则返回 `false`。
>
> `int[] forwardPacket()`：以 FIFO（先进先出）顺序转发下一个数据包。
>
> - 从存储中移除该数据包。
> - 以数组 `[source, destination, timestamp]` 的形式返回该数据包。
> - 如果没有数据包可以转发，则返回空数组。
>
> `int getCount(int destination, int startTime, int endTime)`：
>
> - 返回当前存储在路由器中（即尚未转发）的，且目标地址为指定 `destination` 且时间戳在范围 `[startTime, endTime]`（包括两端）内的数据包数量。
>
> **注意**：对于 `addPacket` 的查询会按照 `timestamp` 的递增顺序进行。

---

难想，需要一个集合去存储包，还需要一个队列去保证 FIFO，为了能够二分查找还需要建立映射关系。基本结构体写好了，剩下的按照逻辑来即可，不想写了，看注释：

```go
type packet struct {
    timestamp int
    destination int
    source int
}

type Router struct {
    memoryLimit int
    // 队列
    packetQ []packet
    // 集合，用于表示是否存在
    packetSet map[packet]struct{}
    // 目的地映射到时间戳的集合，有序。
    destToTimestamps map[int][]int
}


func Constructor(memoryLimit int) Router {
	return Router{
		memoryLimit:      memoryLimit,
		packetSet:        map[packet]struct{}{},
		destToTimestamps: map[int][]int{},
	}
}


func (this *Router) AddPacket(source int, destination int, timestamp int) bool {
    pkt := packet{
        source: source,
        destination: destination,
        timestamp: timestamp,
    }
    // 原本存在，返回 false
    if _, ok := this.packetSet[pkt]; ok {
        return false
    }
    // 加入集合
    this.packetSet[pkt] = struct{}{}
    // 满了
    if len(this.packetQ) == this.memoryLimit {
        this.ForwardPacket()
    }
    // 加入队列
    this.packetQ = append(this.packetQ, pkt)
    // 目的地到时间戳的映射
    this.destToTimestamps[destination] = append(this.destToTimestamps[destination], timestamp)
    return true
}

// 移除逻辑，没啥好说的
func (this *Router) ForwardPacket() []int {
    if len(this.packetQ) == 0 {
        return nil
    }
    pkt := this.packetQ[0]
    this.packetQ = this.packetQ[1:]
    this.destToTimestamps[pkt.destination] = this.destToTimestamps[pkt.destination][1:]
    delete(this.packetSet, pkt)
    return []int{pkt.source, pkt.destination, pkt.timestamp}
}


func (this *Router) GetCount(destination int, startTime int, endTime int) int {
    timestamps := this.destToTimestamps[destination]
    // 二分查找
    return sort.SearchInts(timestamps, endTime + 1) - sort.SearchInts(timestamps, startTime)
}

```

