[679. 24 点游戏](https://leetcode.cn/problems/24-game/)

> 给定一个长度为4的整数数组 `cards` 。你有 `4` 张卡片，每张卡片上都包含一个范围在 `[1,9]` 的数字。您应该使用运算符 `['+', '-', '*', '/']` 和括号 `'('` 和 `')'` 将这些卡片上的数字排列成数学表达式，以获得值24。
>
> 你须遵守以下规则:
>
> - 除法运算符
>
>    
>
>   ```
>   '/'
>   ```
>
>    
>
>   表示实数除法，而不是整数除法。
>
>   - 例如， `4 /(1 - 2 / 3)= 4 /(1 / 3)= 12` 。
>
> - 每个运算都在两个数字之间。特别是，不能使用
>
>    
>
>   ```
>   “-”
>   ```
>
>    
>
>   作为一元运算符。
>
>   - 例如，如果 `cards =[1,1,1,1]` ，则表达式 `“-1 -1 -1 -1”` 是 **不允许** 的。
>
> - 你不能把数字串在一起
>
>   - 例如，如果 `cards =[1,2,1,2]` ，则表达式 `“12 + 12”` 无效。
>
> 如果可以得到这样的表达式，其计算结果为 `24` ，则返回 `true `，否则返回 `false` 。

---

厉害的思路，通过这样dfs，括号的情况也包括进去，好吓人，开背！

```go
func judgePoint24(cards []int) bool {
	SearchCards := make([]float64, len(cards))
	for i := range SearchCards {
		SearchCards[i] = float64(cards[i])
	}
	return dfs(SearchCards)
}

func dfs(nums []float64) bool {
	if len(nums) == 1 {
		return math.Abs(nums[0]-24) < 1e-9
	}
	flag := false
	for i := 0; i < len(nums); i++ {
		for j := i + 1; j < len(nums); j++ {
			n1, n2 := nums[i], nums[j]
			NewNums := make([]float64, 0)
			for k := 0; k < len(nums); k++ {
				if k != i && k != j {
                    //保证i和j不被加入新数组，而之后加入i和j的加减乘除组合
					NewNums = append(NewNums, nums[k])
				}
			}
            //枚举计算情况，并且dfs到下一层，每一层长度-1
			flag = flag || dfs(append(NewNums, n1+n2))
			flag = flag || dfs(append(NewNums, n1-n2))
			flag = flag || dfs(append(NewNums, n2-n1))
			flag = flag || dfs(append(NewNums, n1*n2))
			if n1 != 0 {
				flag = flag || dfs(append(NewNums, n2/n1))
			}
			if n2 != 0 {
				flag = flag || dfs(append(NewNums, n1/n2))
			}
			if flag {
				return true
			}
		}
	}
	return false
}
```

