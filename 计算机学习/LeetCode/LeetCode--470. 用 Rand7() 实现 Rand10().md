[470. 用 Rand7() 实现 Rand10()](https://leetcode.cn/problems/implement-rand10-using-rand7/)

> 给定方法 `rand7` 可生成 `[1,7]` 范围内的均匀随机整数，试写一个方法 `rand10` 生成 `[1,10]` 范围内的均匀随机整数。
>
> 你只能调用 `rand7()` 且不能调用其他方法。请不要使用系统的 `Math.random()` 方法。
>
> 每个测试用例将有一个内部参数 `n`，即你实现的函数 `rand10()` 在测试时将被调用的次数。请注意，这不是传递给 `rand10()` 的参数。

---

没见过这么有意思的题目，byd，思路就是将，1~7平面展开，作为索引，然后拒绝部分的展开结果，取模即可。

```go
func rand10() int {
	for {
		x := rand7()
		y := rand7()
        ans := (x - 1) * 7 + y
		if ans > 40 {
			continue
		}
		return 1 + ans % 10
	}
}
```

----

