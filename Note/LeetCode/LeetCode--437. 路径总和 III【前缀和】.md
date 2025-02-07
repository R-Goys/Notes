[原题链接](https://leetcode.cn/problems/path-sum-iii/description/?envType=study-plan-v2&envId=top-100-liked)
## 前言
感觉这道题思路很有意思，写一下题解，帮助记忆

## 正文
- **题目描述**
  - 给定一个二叉树的根节点 root ，和一个整数 targetSum ，求该二叉树里节点值之和等于 targetSum 的 路径 的数目。

   - 路径 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。	
- **题目分析**
    - 这道题的递归做法比较容易想到，在这里主要写前缀和的做法。
    - 根据题意，我们需要找到**连续**的节点的和等于**目标值**的个数，通过**连续**这一特性，我们应该想到区间，进而可以使用前缀和来做这道题，如何实现呢？
    - 由于节点存在负数值，我们可以选择哈希表来存储**当前路线**上已经遍历到的前缀和的个数，通俗来说就是：每遍历到一个节点，我们就将当前的前缀和加上这个节点的值，得到当前的前缀和，随后在哈希表中，将当前前缀和的值加上一，随后遍历下一个节点。
    - 那么如何判断是否存在目标值呢？很简单，将当前得到的前缀和减去目标值，看看这个差是否存在于哈希表中即可，如果存在，那么我们便将这个值的个数加起来，这样，我们便得到了当前已经遍历到的目标值的个数。
    - **那么问题来了，万一我们遍历其他路线的节点时，哈希表存储的数值怎么办？**
    - 这一点我们就可以采取回溯的办法，具体实现请看代码：
```cpp
class Solution {
public:
    unordered_map<long long, int> pre;
    int pathSum(TreeNode* root, int targetSum) {
        pre[0] = 1;//初始化，如果前缀和就等于目标值，那么也满足条件
        return dfs(root, 0, targetSum);
    }
    int dfs(TreeNode* root, long long cur, int& targetSum) {
        if (root == nullptr) return 0;
        int ret = 0;
        cur += root->val;//加入当前节点的值
        if (pre.count(cur - targetSum)) {//检查是否存在
            ret = pre[cur - targetSum];
        }

        pre[cur]++;//当前前缀和加入哈希表
        ret += dfs(root->right, cur, targetSum);
        ret += dfs(root->left, cur, targetSum);
        pre[cur]--;//回溯
        return ret;
    }
};
```
## 结语
- 以上便是关于这道题前缀和的做法，如果想知道递归是怎么做的，可以点击原链接看看题解哦，很简单的。

