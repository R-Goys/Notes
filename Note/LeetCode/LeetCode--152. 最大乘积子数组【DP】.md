[152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)

----

## 正文

>给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续 子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。
>
>测试用例的答案是一个 **32-位** 整数。

如题给定一个数组，求连续子数组最大乘积

第一次看见这道题的思路是用动态规划，计算前缀乘积，但是这样一来时间复杂度很高，前缀为0以及负数的情况很难处理。

更好的思路应该是同时统计前缀的最大值和最小值，因为最小值在遇到负数之后会变成最大值，而最大值会变成最小值，通过max嵌套来处理前缀为0的情况,下面是未经优化的代码：

```go
vfunc maxProduct(nums []int) int {
    n := len(nums)
    maxF := make([]int, n)
    minF := make([]int, n)
    
    maxF[0] = nums[0]
    minF[0] = nums[0]
    
    for i := 1; i < n; i++ {
        maxF[i] = max(maxF[i-1]*nums[i], max(nums[i], minF[i-1]*nums[i]))
        minF[i] = min(minF[i-1]*nums[i], min(nums[i], maxF[i-1]*nums[i]))
    }
    
    ans := maxF[0]
    for i := 1; i < n; i++ {
        ans = max(ans, maxF[i])
    }
    
    return ans
}
```

时间和空间复杂度均为O(N)，但是我们可以发现，maxF和minF中的每个数据都只是在计算后一个数据的时候才用到了一次，最后则是统计最大值，所以在这里我们可以进行优化，只需要在计算maxF和minF的时候直接计算ans即可，这样就可以将空间复杂度优化为O(1),代码如下：

```go
func maxProduct(nums []int) int {
    maxF, minF, ans := nums[0], nums[0], nums[0]

    for i := 1; i < len(nums); i++ {
        mx, mn := maxF, minF
        maxF = max(mx * nums[i], max(nums[i], mn * nums[i]))
        minF = min(mn * nums[i], min(nums[i], mx * nums[i]))
        ans = max(maxF, ans)
    }

    return ans
}
```



### 二刷dp

没写出来，明明才写过没好久，哎

```go
func maxProduct(nums []int) int {

	n := len(nums)
	MaxF := make([]int, n)
	MinF := make([]int, n)
    ans := nums[0]
	MaxF[0], MinF[0] = nums[0], nums[0]

	for i := 1; i < n; i++ {
		MaxF[i] = max(MaxF[i-1]*nums[i], max(MinF[i-1]*nums[i], nums[i]))
		MinF[i] = min(MinF[i-1]*nums[i], min(MaxF[i-1]*nums[i], nums[i]))
        ans = max(ans, MaxF[i])
	}
    return ans
}
```

**优化细节：**

为什么需要mx，mn变量？

因为再有多个max和min表达式，在计算max，min的时候，会有先后顺序，造成不一致性

```go
func maxProduct(nums []int) int {
	n := len(nums)
	ans, MaxF, MinF := nums[0], nums[0], nums[0]

	for i := 1; i < n; i++ {
        mn, mx := MinF, MaxF
		MaxF = max(mx*nums[i], max(mn*nums[i], nums[i]))
		MinF = min(mn*nums[i], min(mx*nums[i], nums[i]))
        ans = max(ans, MaxF)
	}
    return ans
}
```



----

### 结语

最近决定要稍微多写一点题解了，只要是第一次写错的，争取都写一遍吧, 欢迎交流学习