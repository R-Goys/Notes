[300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

---

## 前言

这道题的优化很不错，于是想来写一下。

## 正文

>给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。
>
>**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

---

O(N^2)的解决方法很简单，这里不多赘述，如下：

```go
func lengthOfLIS(nums []int) int {
    f := make([]int, len(nums) + 1)
    ans := 0
    for i := 0; i < len(nums); i ++ {
        f[i] = 1
        for j := 0; j <= i ; j ++ {
            if nums[i] > nums[j] {
                f[i] = max(f[i], f[j] + 1)
            }
            ans = max(f[i], ans)
        }
    }
    return ans
}
func max(i, j int) int {
    if i < j {
        return j
    }
    return i
}
```

接下来便是我们的优化思路，我们知道，O(N^2)的解决办法就是动态规划，两层for来进行遍历，我们需要知道，哪里出现了多余的计算，使得我们能够把这部分计算优化掉？

比如说, `[1,2,3,4,5,6]` 这个序列，刚好就是一个严格单调序列，因此，对于我们两层的for循环来说就是灾难性的，很明显在第二层for循环中，我们需要不断比较**新插入**的数字和已经遍历的数字进行比较，很容易发现，在这个序列中，直接将新插入的数字直接插入在队尾就行了，无需进行之前冗余的比较。

既然如此，怎么做到省去这些多余的比较呢？这就你理解一下我刚刚提到的**插入**的意思，也就是说，第一层循环保持不变，每遍历到一个数字，就将这个数字插入到**正确的位置**，而这个位置的下标，则是`包含当前插入的数字的最长子序列`简而言之，我们维护的序列并不一定是正确的最长子序列，然而这个序列的`长度`一定是最长子序列的长度，这点需要细细的想一下。，至于**正确的位置**则是将数字插入该位置之后，该序列依旧是单调递增的，因此，我们就可以采取二分查找来找到这个**正确的位置**

代码如下：

```go
func lengthOfLIS(nums []int) int {
    f := make([]int, len(nums) + 1)
    Len := 0
    for i := 0; i < len(nums); i ++ {
        l := 0
        r := Len
        for l < r {
            mid := (l + r + 1) / 2
            if f[mid] < nums[i] {
                l = mid
            } else {
                r = mid - 1
            }
        }
        Len = max(Len, r + 1)
        f[r + 1] = nums[i]
    }
    return Len
}


func max(i, j int) int {
    if i < j {
        return j
    }
    return i
}
```

----

二刷：

```go
func lengthOfLIS(nums []int) int {
    n := len(nums)
    ans := 0
    f := make([]int, n)
    for i := 0; i < n; i ++ {
        f[i] = 1
        for j := 0; j <= i; j ++ {
            if nums[i] > nums[j] {
                f[i] = max(f[i], f[j] + 1)
            }
            ans = max(f[i], ans)
        }
    }
    return ans
}
```

二分优化：

```go
func lengthOfLIS(nums []int) int {
    n := len(nums)
    f := make([]int, n + 1)
    Len := 0
    for i := 0; i < n; i ++ {
        l := 0
        r := Len
        for l < r {
            mid := (l + r + 1) / 2
            if f[mid] < nums[i] {
                l = mid
            } else {
                r = mid - 1
            }
        }
        Len = max(Len, r + 1)
        f[r + 1] = nums[i]
    }
    return Len
}
```

关于这部分，二分边界也是想了很久，详情可以看我的文章： [二分查找边界问题](https://rs-notes.gitbook.io/r/note/suan-fa-za-tan/guan-yu-er-fen-cha-zhao-shi-de-bian-jie-fen-lei-wen-ti) 
