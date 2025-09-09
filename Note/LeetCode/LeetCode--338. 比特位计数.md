[338. 比特位计数](https://leetcode.cn/problems/counting-bits/)

> 给你一个整数 `n` ，对于 `0 <= i <= n` 中的每个 `i` ，计算其二进制表示中 **`1` 的个数** ，返回一个长度为 `n + 1` 的数组 `ans` 作为答案。

---

0人会用暴力，直接根据位运算的特性来做。

```cpp
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> ans(n + 1);
        for (int i = 0; i <= n; i ++) {
            int ret = 0;
            if (i % 2 == 0) {
                ret = ans[i / 2];
            } else {
                ret = ans[i / 2] + 1;
            }
            ans[i] = ret;
        }
        return ans;
    }
};
```

