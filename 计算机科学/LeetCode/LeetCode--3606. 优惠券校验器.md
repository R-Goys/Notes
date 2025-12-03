[3606. 优惠券校验器](https://leetcode.cn/problems/coupon-code-validator/)

> 给你三个长度为 `n` 的数组，分别描述 `n` 个优惠券的属性：`code`、`businessLine` 和 `isActive`。其中，第 `i` 个优惠券具有以下属性：
>
> - `code[i]`：一个 **字符串**，表示优惠券的标识符。
> - `businessLine[i]`：一个 **字符串**，表示优惠券所属的业务类别。
> - `isActive[i]`：一个 **布尔值**，表示优惠券是否当前有效。
>
> 当以下所有条件都满足时，优惠券被认为是 **有效的** ：
>
> 1. `code[i]` 不能为空，并且仅由字母数字字符（a-z、A-Z、0-9）和下划线（`_`）组成。
> 2. `businessLine[i]` 必须是以下四个类别之一：`"electronics"`、`"grocery"`、`"pharmacy"`、`"restaurant"`。
> 3. `isActive[i]` 为 **true** 。
>
> 返回所有 **有效优惠券的标识符** 组成的数组，按照以下规则排序：
>
> - 先按照其 **businessLine** 的顺序排序：`"electronics"`、`"grocery"`、`"pharmacy"`、`"restaurant"`。
> - 在每个类别内，再按照 **标识符的字典序（升序）**排序。

---

没啥技术含量，就是排序

```go
import "sort"
func validateCoupons(code []string, businessLine []string, isActive []bool) []string {
    mp := map[string]int{
        "electronics":1,
        "grocery":2,
        "pharmacy":3,
        "restaurant":4,
    }
    ans := [][2]string{}
    for i := 0; i < len(code); i ++ {
        if isActive[i] == false || code[i] == "" {
            continue
        }
        if _, ok := mp[businessLine[i]]; !ok {
            continue
        }
        ok := false
        for _, x := range code[i] {
            if !((x <= '9' && x >= '0') || (x >= 'A' && x <= 'Z') || (x >= 'a' && x <= 'z') || x == '_') {
                ok = true
                break
            }
        }
        if !ok {
            ans = append(ans, [2]string{code[i], businessLine[i]})
        }
    }
    sort.Slice(ans, func(i, j int) bool {
        return mp[ans[i][1]] < mp[ans[j][1]] || (mp[ans[i][1]] == mp[ans[j][1]] && ans[i][0] < ans[j][0])
    })
    res := []string{}
    for _, x := range ans {
        res = append(res, x[0])
    }
    return res
}
```

