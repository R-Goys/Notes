﻿### [LeetCode-33.搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)

---

#### 前言

关于这道题，我最开始想把这个旋转数组还原回去，但是后来发现没有那么麻烦，直接二分即可，重点在于关于当前区间的有序判断，故来写一份题解来分享一下。

#### 正文

首先我们看看题目要我们干什么，题目大意就是给定一个经过轮转的有序数组和一个target值，要我们找到这个target在数组中的下标，没有则返回-1.

既然有序，那么便能和二分扯上关系，但是这个数组经过了轮转，这就很有意思了，又要二分，又不是完全的有序，怎么解决呢？这需要我们利用这个数组的**一些性质**，对二分的每一步进行合理的条件判断。

首先第一步，我们通过`(L + R) / 2`找到的中间值所在的位置的判断，通过这个题目的意思我们可以知道，nums[0]一定大于所有的被旋转到右边的数，一定小于除了这些数字以外的所有数字，姑且可以把他看成是一个类二叉树的"根右左"遍历，这样主要是可以更好的理解

但并不是二叉排序树，这个懂得都懂

紧接着我来谈谈第一步怎么判断，如果 `nums[mid] > nums[0]` 的话，那么可以确定，`mid` 存在于这个根的右边，也就是大于 `nums[0]` 的值，也可以说成是在数组中， `0-mid` 是严格递增的，

随后我们进行下一步判断，当 `nums[mid] > nums[0]` 的时候，此时我们需要缩小我们的查找范围，也就是改变L或者R的值，这也是一个问题，首先我们要想一想，mid是在哪一个区间里面？如何改变L会有什么影响？改变R会有什么影响？

此时如果我们仅仅根据 `nums[mid]` 和 `target` 的大小关系直接进行判断肯定是不行的，于是，我们还需要加上一层target所在区间的判断，这个是跟第一步的判断类似

如果 `target > nums[0]` 那么说明下标为0的点到target这一区间的数字也是严格递增的，此时如果target值在[l, r]的左半区间，便可以直接将我们的 `r` 赋值为 `mid - 1`, 否则将 `l` 赋值为 `mid + 1`,为什么可以直接判断应该赋值为mid + 1呢？下面细说。

如果 `0 ~ target` 他并不是严格递增的区间，怎么办？这恰恰说明了target处于[l, r]的右半区间，因为此时 `nums[mid]` 他是处于中间位置的，并且处于有序区间上，那么target必然是在 `nums[mid]`的右边，于是乎我们便可以直接将 `l` 赋值为 `mid + 1`,其他的就正常判断就好了。

-----

下面说说nums[0] > nums[mid]的情况

其实思路都是差不多的，此时仍需要判断target的位置，此时若target小于nums[0]那么则说明了target也不在严格递增的数组区间上，那么此时，如果target也在mid的右侧，则可以直接将 `l` 赋值为 `mid + 1`,否则，将 `r` 赋值为 `mid - 1`，这里思路和上面一致。

下面就是具体实现的代码【Golang】

```go
func search(nums []int, target int) int {
    n := len(nums)
    l := 0
    r := n - 1
    var mid int
    for l <= r {
        mid = (l + r) / 2
        if nums[mid] == target {
            return mid
        }
        if nums[0] <= nums[mid] {
            if nums[0] <= target && nums[mid] > target {
                r = mid - 1
            } else {
                l = mid + 1
            }
        } else {
            if nums[mid] < target && target < nums[0] {
                l = mid + 1
            } else {
                r = mid - 1
            }
        }
    }
    return -1
}
```

### 二刷

```go
func search(nums []int, target int) int {
    mid, l, r := 0, 0, len(nums) - 1
    
    for l <= r {
        mid = (l + r) / 2
        if nums[mid] == target {
            return mid
        }
        
        if nums[0] <= nums[mid] {
            if nums[0] <= target && nums[mid] > target {
                r = mid - 1
            } else {
                l = mid + 1
            }
        } else {
            if nums[mid] < target && nums[0] > target {
                l = mid + 1
            } else {
                r = mid - 1
            }
        }
    }
    return -1
}
```



---

#### 结语

这道题到这里就结束了，如果还是没明白，或者有什么问题，欢迎给我留言！
