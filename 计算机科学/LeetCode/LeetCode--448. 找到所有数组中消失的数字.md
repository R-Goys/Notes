[448. 找到所有数组中消失的数字](https://leetcode.cn/problems/find-all-numbers-disappeared-in-an-array/)

> 给你一个含 `n` 个整数的数组 `nums` ，其中 `nums[i]` 在区间 `[1, n]` 内。请你找出所有在 `[1, n]` 范围内但没有出现在 `nums` 中的数字，并以数组的形式返回结果。

---

哈希表

```cpp
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
        vector<int> ans;
        unordered_set<int> mp;
        for (int x : nums) {
            mp.insert(x);
        }

        for (int i = 1; i <= nums.size(); i ++) {
            if (mp.find(i) == mp.end()) {
                ans.push_back(i);
            }
        }
        return ans;
    }
};
```

