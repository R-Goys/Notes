[416. 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)

[494. 目标和](https://leetcode.cn/problems/target-sum/)

----

## 前言

哈哈，又是背包问题，一开始没写出来，写个题解加深记忆

## 正文

### 416. 分割等和子集

> 给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

如题，给定一个数组，求是否能将这个数组划分为和相等的两部分。

乍一看，首先我们需要知道，由于数组中只含整数类型，所以这个数组的**Sum**不能为奇数，并且当数组长度小于2时，也无法进行分割，所以满足这些条件的直接返回false即可。

接下来，题目要求我们求是否能将数组划分为和相等的两部分，其实就是求数组中是否存在和为**Sum/2**的子集(不需要连续)。

既然如此，就可以转换成我们的01背包问题，每遍历到一个“物品”，就需要判断选还是不选这个物品加入背包，我们知道，我们只需要判断**是否**存在的问题，所以只需要根据上一个状态来进行**逻辑或**计算即可，所以代码如下：

```go
func canPartition(nums []int) bool {
    sum := 0
    for i := 0; i < len(nums); i ++ {
        sum += nums[i]
    }
    if sum % 2 == 1 || len(nums) < 2 {
        return false
    }

    V := sum / 2
    f := make([]bool, V + 1)
    f[0] = true

    for i := 0; i < len(nums); i++ {
        for j := V; j >= nums[i]; j-- {
            f[j] = f[j] || f[j-nums[i]]
        }
    }
    return f[V]
}
```

---

二刷背包，记得反向遍历

```go
func canPartition(nums []int) bool {
    sum := 0
    n := len(nums)
    var target int
    for i := 0; i < n; i ++ {
        sum += nums[i]
    }
    if sum % 2 == 1 {
        return false
    }
    target = sum / 2
    f := make([]bool, target + 1)
    f[0] = true
    for i := 1; i <= n; i ++ {
        for j := target; j >= 1; j -- {
            if j - nums[i - 1] >= 0 {
                f[j] =  f[j] || f[j - nums[i - 1]]
            }
        }
    }
    return f[target]
}
```



### 494. 目标和

>给你一个非负整数数组 `nums` 和一个整数 `target` 。
>
>向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：
>
>- 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。
>
>返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

rt，给定一个数组和目标值**target**，要求数组中的所有数字都要用到，乍一看下不了手，但是我们可以将数组中每个数加起来得到**Sum**，然后减去给定的目标值**target**除以2，就得到了所有符号为'-'的数字的和了,想要知道这里是怎么来的，需要建立一个方程:

>符号为正的数 - 符号为负的数 = target
>
>符号为正的数 + 符号为负的数 = sum

联立求解，就是得到了符号相同的数字的和，这里还可以进一步优化，就是给target加上绝对值。

随后就是和上面一样的背包问题，代码如下：

```go
func findTargetSumWays(nums []int, target int) int {
    sum := 0
    for i := 0; i < len(nums); i++ {
        sum += nums[i]
    }
    sum -= target
    if sum < 0 || sum % 2 == 1 {
        return 0
    }
    V := sum / 2
    f := make([]int, V + 1)
    f[0] = 1


    for i := 0; i < len(nums); i ++ {
        for j := V; j >= nums[i]; j -- {
            f[j] += f[j - nums[i]]
        }
    }
    return f[V]
}
```

---

二刷目标和，这个做减法的思路还是没回忆起来，我去

```go
func findTargetSumWays(nums []int, target int) int {
    sum := 0
    n := len(nums)
    for i := 0; i < n; i ++ {
        sum += nums[i]
    }
    v := sum - target
    if v < 0 || v % 2 == 1 {
        return 0
    }
    v /= 2
    f := make([]int, v + 1)
    f[0] = 1
    for i := 1; i <= n ; i ++ {
        for j := v; j >= nums[i - 1]; j -- {
            if j - nums[i - 1] >= 0 {
                f[j] += f[j - nums[i - 1]]
            }
        }
    }
    return f[v]
}
```



## 结语

菜就多练~
