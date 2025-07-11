[76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

> 给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。

----

最开始只会暴力滑动窗口做法：O（M*N）或者O（c\N+M）

然后怎么都想不出来怎么优化到O（M + N）

于是去看了0x3f的题解，恍然大悟！

下面来讲讲怎么做的：

首先创建一个mp，统计target目标字符串的字母数量，再用一个cnt变量存储存在的字符种类，

然后用变量AnsL，AnsR，l，r分别表示最终的左右边界，和临时遍历的左右边界，随后在遍历的时候，我们要做以下处理：

1. 每遍历到一个字符，令其在mp中的值-1，如果这个mp的数量变成0，则令cnt的数量-1
2. 随着不断地遍历字符串s，cnt的值最终会变成0，此时我们可以做我们想要的处理了
3. 首先如果当前存储的AnsL和AnsR的长度大于临时遍历的l和r，我们可以更新我们的AnsL，和AnsR
4. 然后考虑左边界是否移动的问题，由于此时不更新左边界的话，之后一定无法找到更小的区间，所以我们还需要考虑左边界的移动，移动左边界需要做的处理有：左边界此时的对应的mp值 + 1，左边界自增，如果当前mp值 == 0，令cnt，字符串种类 ++，此时就需要跳出循环了

值得一提的是，虽然s可能会引入新的字符到哈希表里面，但是实际上是不会造成什么影响的，在哈希表中，正数表示l，r维护的字符串还需要这个字符，而0，表示此时刚好满足这个字符的需求，负数表示可有可无。如果引入了另外的字符，那么之后他不再有可能遍历到使它对应的mp值为0的位置。

```go
func minWindow(s string, t string) string {
    mp := make(map[byte]int, 1)
    cnt := 0
    for i, _ := range t {
        if mp[t[i]] == 0 {
            cnt ++
        }
        mp[t[i]]++
    }

    AnsL, AnsR, l, r := -1, len(s) - 1, 0, 0
    
    for r = 0; r < len(s); r ++ {
        mp[s[r]] --
        if mp[s[r]] == 0 {
            cnt --
        }

        for cnt == 0 {
            if r - l < AnsR - AnsL {
                AnsL, AnsR = l, r
            }

            if mp[s[l]] == 0 {
                cnt ++
            }
            mp[s[l]]++
            l ++
        }
    }
    if AnsL < 0 {
        return ""
    }
    return s[AnsL:AnsR + 1]
}
```

二刷：

```go
func minWindow(s string, t string) string {
    nums := ['z' + 1]int{}
    all := 0
    for _, x := range t {
        if nums[x] == 0 {
            all ++
        }
        nums[x] ++
    }
    lans := -1
    rans := 0x3f3f3f3f
    l := 0
    for r, x := range s {
        nums[x] --
        if nums[x] == 0 {
            all --
        }
        for all == 0 {
            if rans - lans + 1 > r - l + 1 {
                rans = r
                lans = l
            }
            if nums[s[l]] == 0 {
                all ++
            }
            nums[s[l]] ++
            l ++
        }
    }
    if rans == 0x3f3f3f3f {
        return ""
    }
    return s[lans : rans + 1]
}
```

