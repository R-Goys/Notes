[394. 字符串解码](https://leetcode.cn/problems/decode-string/)

----

## 前言

这道题跟表达式求值有点类似，费了点功夫，故来写一下题解

----

## 正文

>给定一个经过编码的字符串，返回它解码后的字符串。
>
>编码规则为: `k[encoded_string]`，表示其中方括号内部的 `encoded_string` 正好重复 `k` 次。注意 `k` 保证为正整数。
>
>你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
>
>此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 `k` ，例如不会出现像 `3a` 或 `2[4]` 的输入

题目如上，大意就是给定一个字符串，其中有格式为num [str]的字符串，我们需要将这些格式的字符串变换为num * str格式的字符串，就是重复了num次的str，由于给定了''[]'来帮助我们定位，根据经典的表达式求值这道题，我们可以联想到通过栈来解决，需要注意的是，数字的范围是1~300。

在题目中最为不好处理的就是嵌套括号的情况，下面我们来分析一下，以`3[a2[c]]`为例子，为方便处理，我们将数字和字母分开放在栈中，当我们读取完3之后，遍历到了 `[` 此时就需要进行处理，将后续遍历到的字符串`a`放入栈中，然后继续遍历数字2,并放入数字栈中，然后又遍历到了 `[`，此时再将后续遍历的字符串 `c`放入字符串栈中，然后遇到了 `]` 此时我们就将栈中元素弹出，数字栈弹出的为 `2` ，字符串栈弹出的为 `c`  ,然后此时我们得到了 `cc`，但是这并不是答案，此时我们还需要拿到原本字符串栈里面的前一个元素 `a`，来进行拼接，最后得到了我们的`acc`然后弹出数字栈的元素，得到了我们的最终答案，最后的栈顶元素便是我们的答案。

```go
func decodeString(s string) string {
    var nums []int
    var strs []string
    i := 0

    for i < len(s) {
        if s[i] >= '0' && s[i] <= '9' {
            num := 0
            // 读取完整的数字（数字可以有多位）
            for i < len(s) && s[i] >= '0' && s[i] <= '9' {
                num = num*10 + int(s[i]-'0')
                i++
            }
            nums = append(nums, num)
        } else if s[i] == '[' {
            strs = append(strs, "") // 新建一个空字符串，用来保存当前括号内的内容
            i++
        } else if s[i] == ']' {
            // 当遇到 ] 时，开始解码
            num := nums[len(nums)-1]
            nums = nums[:len(nums)-1]

            // 获取当前括号内的内容
            str := strs[len(strs)-1]
            strs = strs[:len(strs)-1]

            // 连接之前的字符串
            var pre string
            if len(strs) > 0 {
                pre = strs[len(strs)-1]
                strs = strs[:len(strs)-1]
            } else {
                pre = ""
            }

            // 将当前括号内的字符串重复 num 次
            for k := 0; k < num; k++ {
                pre += str
            }

            // 将结果添加到 strs 中
            strs = append(strs, pre)
            i++
        } else {
            // 处理字母字符，直接加到当前字符串
            if len(strs) == 0 {
                strs = append(strs, "") // 确保 strs 不为空
            }
            strs[len(strs)-1] += string(s[i])
            i++
        }
    }
    
    return strs[0]
}

```

----

## 二刷

还是遇到了一些问题，距离上次做隔了一个多月，忘了很多。

```go
func decodeString(s string) string {
    var strs []string
    var nums []int
    i := 0
    for i < len(s) {
        if s[i] <= '9' && s[i] >= '0' {
            num := 0
            for s[i] <= '9' && s[i] >= '0' && i < len(s) {
                num = num * 10 + int(s[i] - '0')
                i ++
            }
            nums = append(nums, num)
        } else if s[i] == '[' {
            strs = append(strs, "")
            i ++
        } else if s[i] == ']' {
            num := nums[len(nums) - 1]
            nums = nums[:len(nums) - 1]

            str := strs[len(strs) - 1]
            strs = strs[:len(strs) - 1]

            pre := ""
            if len(strs) > 0 {
                pre = strs[len(strs) - 1]
                strs = strs[:len(strs) - 1]
            }
            for j := 0; j < num; j ++ {
                pre += str
            }
            strs = append(strs, pre)
            i++
        } else {
            if len(strs) == 0 {
                strs = append(strs, "")
            }
            strs[len(strs) - 1] += string(s[i])
            i ++
        }
    }
    return strs[0]
}
```

