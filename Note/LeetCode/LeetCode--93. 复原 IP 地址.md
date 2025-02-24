[93. 复原 IP 地址](https://leetcode.cn/problems/restore-ip-addresses/)

> **有效 IP 地址** 正好由四个整数（每个整数位于 `0` 到 `255` 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。
>
> - 例如：`"0.1.2.201"` 和` "192.168.1.1"` 是 **有效** IP 地址，但是 `"0.011.255.245"`、`"192.168.1.312"` 和 `"192.168@1.1"` 是 **无效** IP 地址。
>
> 给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能的**有效 IP 地址**，这些地址可以通过在 `s` 中插入 `'.'` 来形成。你 **不能** 重新排序或删除 `s` 中的任何数字。你可以按 **任何** 顺序返回答案。

----

### dfs

第一次看到，像是dfs，一做，还真是，不过要处理的情况有很多，比如前导零，'.'的个数，遍历的起点。

首先，用ans存储string的切片，通过传递pointNum来知道当前小数点的个数，通过st来知道当前遍历的起点，ip来存储当前计算出来的ip。

首先，当pointNum == 4时，判断字符串长度是否等于st，如果是，存入当前ip，如果不是，直接返回，至于ip合理性的判断，是在计算ip的时候就已经做了判断，这里不需要多余的判断，下面会提到

然后如果当前st != 0,那么在ip后面加上"."

然后就是激动人心的遍历字符串了，从st开始遍历，通过最多遍历三个字符的条件来进行一定程度的剪枝，然后如果当前数字具有前导零就是说`num[0] == '0'`的话，就continue，不断continue直到数字没有前导零。

如果没有前导零，将字符串转换为int，判断是否有效，有效则加入ip，进行下一轮dfs，无效则继续遍历

值得注意的是，dfs之后需要恢复ip的状态。

```go
func restoreIpAddresses(s string) []string {
    var ans []string
    dfs(&ans, s, "", 0, 0)
    return ans
}

func dfs(ans *[]string, s string, ip string, st int, pointNum int) {
    if pointNum == 4 {
        if st == len(s) { 
            *ans = append(*ans, ip)
        }
        return
    }

    if st != 0 {
        ip += "."
    }
    var num string
    for i := st; i < len(s) && i < st + 3; i ++ {
        num = s[st : i + 1]
        if len(num) > 1 && num[0] == '0' {
            continue
        }
        if x,_ := strconv.Atoi(num); x >= 0 && x <= 255 {
            ip += num
            dfs(ans, s, ip, i + 1, pointNum + 1)
            ip = ip[:len(ip) - len(num)]
        }
    }
}
```

