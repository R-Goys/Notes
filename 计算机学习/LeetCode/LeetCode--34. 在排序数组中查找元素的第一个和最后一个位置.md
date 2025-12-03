[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

> 给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。
>
> 如果数组中不存在目标值 `target`，返回 `[-1, -1]`。
>
> 你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题

----

二分边界练习题，关于左右边界查找的二分问题可以看看我的算法杂谈--二分

```go
func searchRange(nums []int, target int) []int {
    if len(nums) == 0 {
        return []int{-1, -1}
    }
    l, r := 0, len(nums) - 1
    for l < r {
        mid := (l + r) / 2
        if nums[mid] >= target {
            r = mid
        } else {
            l = mid + 1
        }
    }
    if nums[l] != target {
        return []int{-1, -1}
    }
    Left := l
    l, r = 0, len(nums) - 1
    for l < r {
        mid := (l + r + 1) / 2
        if nums[mid] <= target {
            l = mid
        } else {
            r = mid - 1
        }
    }
    return []int{Left, r}
}
```

二刷，凭借直觉直接秒

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int l = 0, r = nums.size() - 1;
        while (l < r) {
            int mid = (l + r) / 2;
            if (nums[mid] < target) {
                l = mid + 1;
            } else {
                r = mid;
            }
        }
        int left = r;
        l = 0, r = nums.size() - 1;
        while (l < r) {
            int mid = (l + r + 1) / 2;
            if (nums[mid] <= target) {
                l = mid;
            } else {
                r = mid - 1;
            }
        }
        int right = r;
        if (left < 0 || right >= nums.size() || nums[right] != target && nums[left] != target) {
            return {-1, -1};
        }

        return {left, right};
    }
};
```

