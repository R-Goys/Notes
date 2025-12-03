[2431. 最大限度地提高购买水果的口味](https://leetcode.cn/problems/maximize-total-tastiness-of-purchased-fruits/)

> 你有两个非负整数数组 `price` 和 `tastiness`，两个数组的长度都是 `n`。同时给你两个非负整数 `maxAmount` 和 `maxCoupons`。
>
> 对于范围 `[0, n - 1]` 中的每一个整数 `i`:
>
> - `price[i]` 描述了第 `i` 个水果的价格。
> - `tastiness[i]` 描述了第 `i` 个水果的味道。
>
> 你想购买一些水果，这样总的味道是最大的，总价不超过 `maxAmount`。
>
> 此外，你还可以用优惠券以 **半价** 购买水果 (向下取整到最接近的整数)。您最多可以使用 `maxCoupons` 次该优惠券。
>
> 返回可购买的最大总口味。
>
> **注意:**
>
> - 每个水果最多只能购买一次。
> - 一个水果你最多只能用一次折价券。

---

还是背包问题，不过有一些条件需要特判，比如价格为 0 的时候，如果在价格为 0 的时候，加上优惠券就会导致重复计算，答案就会错。
```go
func maxTastiness(price []int, tastiness []int, maxAmount int, maxCoupons int) int {
    n := len(price)
    f := make([][]int, maxCoupons + 1)
    for i := 0; i <= maxCoupons; i ++ {
        f[i] = make([]int, maxAmount + 1)
    }
    for i := 0; i < n; i ++ {
        for j := maxAmount; j >= price[i] / 2; j -- {
            for k := maxCoupons; k >= 0; k -- {
                if j >= price[i] / 2 && k != 0 && price[i] != 0 {
                    f[k][j] = max(f[k][j], f[k - 1][j - price[i] / 2] + tastiness[i])
                }
                if j >= price[i] {
                    f[k][j] = max(f[k][j], f[k][j - price[i]] + tastiness[i])
                }
            }
        }
    }
    return f[maxCoupons][maxAmount]
}
```

