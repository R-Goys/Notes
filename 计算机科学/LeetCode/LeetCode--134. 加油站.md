[134. 加油站](https://leetcode.cn/problems/gas-station/)

> 在一条环路上有 `n` 个加油站，其中第 `i` 个加油站有汽油 `gas[i]` 升。
>
> 你有一辆油箱容量无限的的汽车，从第 `i` 个加油站开往第 `i+1` 个加油站需要消耗汽油 `cost[i]` 升。你从其中的一个加油站出发，开始时油箱为空。
>
> 给定两个整数数组 `gas` 和 `cost` ，如果你可以按顺序绕环路行驶一周，则返回出发时加油站的编号，否则返回 `-1` 。如果存在解，则 **保证** 它是 **唯一** 的。

---

加油站，贪心思想

从第一个加油站开始，计算将获得的油和消耗的油的差值，并加入balance中，也就是油量的累计变化量，而minBalance记录的是之前记录过的最小的持有的油量，如果小于这个值，则需要更新我们的minBalance和minIndex，因为此时的油量是最小的，可能无法前往下一个节点，此时，此时需要更新下一个节点为起点。

而最后，如果总共累积的油量>=0，则说明可以环绕一圈，如果不是，则不行。

感觉背一背，理解理解就差不多了。

```go
func canCompleteCircuit(gas []int, cost []int) int {
    balance := 0
    minIndex, minBalance := 0, 0
    for i := 0; i < len(gas); i ++ {
        balance += gas[i] - cost[i]
        if balance < minBalance {
            minBalance = balance
            minIndex = i + 1
        }
    }
    if balance >= 0 {
        return minIndex
    }
    return -1
}
```

