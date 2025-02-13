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