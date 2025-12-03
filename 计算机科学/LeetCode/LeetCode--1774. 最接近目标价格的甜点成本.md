[1774. 最接近目标价格的甜点成本](https://leetcode.cn/problems/closest-dessert-cost/)

> 你打算做甜点，现在需要购买配料。目前共有 `n` 种冰激凌基料和 `m` 种配料可供选购。而制作甜点需要遵循以下几条规则：
>
> - 必须选择 **一种** 冰激凌基料。
> - 可以添加 **一种或多种** 配料，也可以不添加任何配料。
> - 每种类型的配料 **最多两份** 。
>
> 给你以下三个输入：
>
> - `baseCosts` ，一个长度为 `n` 的整数数组，其中每个 `baseCosts[i]` 表示第 `i` 种冰激凌基料的价格。
> - `toppingCosts`，一个长度为 `m` 的整数数组，其中每个 `toppingCosts[i]` 表示 **一份** 第 `i` 种冰激凌配料的价格。
> - `target` ，一个整数，表示你制作甜点的目标价格。
>
> 你希望自己做的甜点总成本尽可能接近目标价格 `target` 。
>
> 返回最接近 `target` 的甜点成本。如果有多种方案，返回 **成本相对较低** 的一种。

---

万物皆可 01 背包☠️

第一次看没写出来，看了官解，写了一遍，感觉勉强理解了，主要思路就是将先把所有基料的价格状态设置为 true，然后遍历配料价格进行状态转移，超过 target 的价格和小于 target 的价格分开维护，这里就是简单的 01 背包问题了。

姑且写个注释

```go
func closestCost(baseCosts []int, toppingCosts []int, target int) int {
    x := baseCosts[0]
    for _, c := range baseCosts {
        x = min(x, c)
    }
    // 如果最小基础价格就已经超过 target，直接返回它
    if x > target {
        return x
    }

    // can[i] 表示是否可以恰好凑出价格 i（i <= target）
    can := make([]bool, target+1)
    // ans 用来记录超过 target 的最小可行价格
    ans := 2 * target - x

    // 把所有不超过 target 的基础价格标记为 true
    for _, c := range baseCosts {
        if c <= target {
            can[c] = true
        } else {
            // 更新大于 target 的最小值
            ans = min(ans, c)
        }
    }

    // 对于每种配料，最多可以选 2 次
    for _, c := range toppingCosts {
        for count := 0; count < 2; count ++ {
            for i := target; i > 0; i -- {
                // 如果原来可以凑出 i，再加上当前配料 c 超过 target，就更新 ans
                if can[i] && i + c > target {
                    ans = min(ans, i + c)
                }
                // 如果 i - c >= 0，并且之前能凑出 i - c，那么现在也可以凑出 i
                if i - c > 0 {
                    can[i] = can[i] || can[i - c]
                }
            }
        }
    }

    // 在不超过 target 的范围内，寻找最接近 target 的价格
    // i 代表与 target 的差值，从小到大遍历
    for i := 0; i <= ans - target; i ++ {
        if can[target -i] {
            // 找到最接近但不超过 target 的值，直接返回
            return target - i
        }
    }

    // 如果所有不超过 target 的组合都不够接近，则返回第一个超过 target 的最小值
    return ans
}
```

