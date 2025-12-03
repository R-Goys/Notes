[825. 适龄的朋友](https://leetcode.cn/problems/friends-of-appropriate-ages/)

> 在社交媒体网站上有 `n` 个用户。给你一个整数数组 `ages` ，其中 `ages[i]` 是第 `i` 个用户的年龄。
>
> 如果下述任意一个条件为真，那么用户 `x` 将不会向用户 `y`（`x != y`）发送好友请求：
>
> - `ages[y] <= 0.5 * ages[x] + 7`
> - `ages[y] > ages[x]`
> - `ages[y] > 100 && ages[x] < 100`
>
> 否则，`x` 将会向 `y` 发送一条好友请求。
>
> 注意，如果 `x` 向 `y` 发送一条好友请求，`y` 不必也向 `x` 发送一条好友请求。另外，用户不会向自己发送好友请求。
>
> 返回在该社交媒体网站上产生的好友请求总数。

---

滑动窗口类型题目，不过要看出对什么进行滑窗，如何滑窗还是很困难的，这里我们需要对年龄进行滑窗，题目中一句话：

> 注意，如果 `x` 向 `y` 发送一条好友请求，`y` 不必也向 `x` 发送一条好友请求。

说实话挺绕的，意思应该是，如果只是单方面满足条件，只能发送一次，如果两边都满足加好友的条件，就可以互相发，根据测试样例应该是这样的。

统计每个年龄对应的人数，Y 的年龄作为左边界，X 的年龄作为右边界，cntWindow 统计总人数，然后进行滑窗就可以了：

```go
func numFriendRequests(ages []int) int {
    cnt := [121]int{}
    for _, x := range ages {
        cnt[x] ++
    }
    ans := 0
    cntWindow, ageY := 0, 0
    for ageX, c := range cnt {
        cntWindow += c
        for ageY * 2 <= ageX + 14 {
            cntWindow -= cnt[ageY]
            ageY ++
        }
        if cntWindow > 0 {
            ans += c * cntWindow - c
        }
    }
    return ans
}
```

