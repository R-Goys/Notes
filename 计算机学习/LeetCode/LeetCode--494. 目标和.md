[494. 目标和](https://leetcode.cn/problems/target-sum/)

> 给你一个非负整数数组 `nums` 和一个整数 `target` 。
>
> 向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：
>
> - 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。
>
> 返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

---

目标和，可以用 offset + target 硬做，更推荐使用特殊的方法，nums 中所有的数字都需要用上，那么所有数字之和再减去 target 代表使用 减法的数字的两倍，这正好说明我们可以将其分为两份，这样就是一个简单的背包问题了，只需要查找 nums 里面有多少种数字可以组成 v 即可。

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int limit = 0;
        for (int x : nums) {
            limit += x;
        }
        int v = limit - target;
        if (v < 0 || v % 2 == 1) {
            return 0;
        }
        v /= 2;
        vector<int> f(v + 1);

        f[0] = 1;
        for (int x : nums)
            for (int j = v; j >= x; j --)
                f[j] += f[j - x];

        return f[v];
    }
};
```

暴力方法也行，但是明显也比较复杂，不推荐，通过偏移量来抵消复数的 target。

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int n = nums.size();
        int limit = 0;
        for (int i = 0; i < n; i ++) {
            limit += nums[i];
        }
        if (target > limit || target < -limit) return 0;
        int offset = limit;
        int m = 2 * limit;
        vector<vector<int>> f(n + 1, vector<int>(m + 1));

        f[0][offset] = 1;
        for (int i = 1; i <= n; i ++ ) {
            for (int sum = -limit; sum <= limit; sum++) {
                int idx = sum + offset;
                if (f[i-1][idx] > 0) {
                    f[i][idx + nums[i - 1]] += f[i - 1][idx];
                    f[i][idx - nums[i - 1]] += f[i - 1][idx];
                }
            }
        }
        return f[n][target + offset];
    }
};
```

