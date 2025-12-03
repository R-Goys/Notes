[169. 多数元素](https://leetcode.cn/problems/majority-element/)

---

### Boyer-Moore 投票算法

没听说过，真是小🔪拉屁股了

```go
func majorityElement(nums []int) int {
    count := 0
    num := -1
    for i := 0; i < len(nums); i ++ {
        if nums[i] == num {
            count ++
        } else {
            count --
            if count < 0 {
                num = nums[i]
                count = 1
            }
        }
    }
    return num
}
```

### 哈希表

简单明了

```go
func majorityElement(nums []int) int {
    mp := make(map[int]int)
    for i := 0; i < len(nums); i++ {
        mp[nums[i]] ++
        if mp[nums[i]] > len(nums)/2 {
            return nums[i]
        }
    }
    return nums[len(nums)]
}
```

另外还可以使用排序来做，不过感觉不如以上两种~



## 二刷

第一次还是只会哈希表：

```go
func majorityElement(nums []int) int {
    n := len(nums)
    mp := make(map[int]int)
    for i := 0; i < n; i ++ {
        mp[nums[i]]++
        if mp[nums[i]] > n / 2 {
            return nums[i]
        }
    }
    return -1
}
```

投票算法，虽然又是看了题解，但是加深了理解，仅对当前出现的数字计数，如果遇见不同的数字，计数-1，如果遇见相同的数字，计数+1，当计数小于0时，说明这个数字的数量至少是小于1/2的，所以排除，但反过来也能说明，如果当前数字没有被覆盖，就说明计数是大于1/2的，总之比较开眼。

```go
func majorityElement(nums []int) int {
    cnt := 0
    num := -1
    for i := 0; i < len(nums); i ++ {
        if num == nums[i] {
            cnt ++
        } else {
            cnt --
            if cnt < 0 {
                num = nums[i]
                cnt = 1
            }
        }
    }
    return num
}
```

