[468. 验证IP地址](https://leetcode.cn/problems/validate-ip-address/)

> 给定一个字符串 `queryIP`。如果是有效的 IPv4 地址，返回 `"IPv4"` ；如果是有效的 IPv6 地址，返回 `"IPv6"` ；如果不是上述类型的 IP 地址，返回 `"Neither"` 。
>
> **有效的IPv4地址** 是 `“x1.x2.x3.x4”` 形式的IP地址。 其中 `0 <= xi <= 255` 且 `xi` **不能包含** 前导零。例如: `“192.168.1.1”` 、 `“192.168.1.0”` 为有效IPv4地址， `“192.168.01.1”` 为无效IPv4地址; `“192.168.1.00”` 、 `“192.168@1.1”` 为无效IPv4地址。
>
> **一个有效的IPv6地址** 是一个格式为`“x1:x2:x3:x4:x5:x6:x7:x8”` 的IP地址，其中:
>
> - `1 <= xi.length <= 4`
> - `xi` 是一个 **十六进制字符串** ，可以包含数字、小写英文字母( `'a'` 到 `'f'` )和大写英文字母( `'A'` 到 `'F'` )。
> - 在 `xi` 中允许前导零。
>
> 例如 `"2001:0db8:85a3:0000:0000:8a2e:0370:7334"` 和 `"2001:db8:85a3:0:0:8A2E:0370:7334"` 是有效的 IPv6 地址，而 `"2001:0db8:85a3::8A2E:037j:7334"` 和 `"02001:0db8:85a3:0000:0000:8a2e:0370:7334"` 是无效的 IPv6 地址。

---

这道题比较无脑，就是要仔细检查特殊字符，字符长度，切片长度等等，以及前导零

```go
func validIPAddress(queryIP string) string {
    if strings.Contains(queryIP, ":") {
        //检查有几段
        slice := strings.Split(queryIP, ":")
        if len(slice) != 8 {
            return "Neither"
        }
        for i := range slice {
            if len(slice[i]) > 4 {
                return "Neither"
            }
            if _, err := strconv.ParseUint(slice[i], 16, 64); err != nil {
                return "Neither"
            }
        }
        return "IPv6"
    } else if strings.Contains(queryIP, ".") {
        //分离检查分段长度
        slice := strings.Split(queryIP, ".")
        if len(slice) != 4 {
            return "Neither"
        }
        for i := range slice {
            if len(slice[i]) > 1 && slice[i][0] == '0' {
                return "Neither"
            }
            if v, err := strconv.Atoi(slice[i]); err != nil || v > 255 {
                return "Neither"
            }
        }
        return "IPv4"
    }
    return "Neither"
}
```

