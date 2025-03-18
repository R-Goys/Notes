[718. 最长重复子数组](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)

> 给两个整数数组 `nums1` 和 `nums2` ，返回 *两个数组中 **公共的** 、长度最长的子数组的长度* 。

子数组，说明连续，如果使用**dp**，整体思路还是和最长公共子序列是一样的，但是有细微的差别

```go
func findLength(nums1 []int, nums2 []int) int {
    n := len(nums1)
    m := len(nums2)
    ans := 0
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            f[i][j] = 0
            if nums1[i - 1] == nums2[j - 1] {
                f[i][j] = f[i - 1][j - 1] + 1
            }
            ans = max(ans, f[i][j])
        }
    }
    return ans
}
```

滚动数组

```go
func findLength(nums1 []int, nums2 []int) int {
    n := len(nums1)
    m := len(nums2)
    ans := 0
    f := make([][]int, 2)
    for i := 0; i < 2; i ++ {
        f[i] = make([]int, m + 1)
    }
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            f[i % 2][j] = 0
            if nums1[i - 1] == nums2[j - 1] {
                f[i%2][j] = f[(i - 1)%2][j - 1] + 1
            }
            ans = max(ans, f[i%2][j])
        }
    }
    return ans
}
```

**字符串哈希**

每次二分表示选择的长度，每次check表示当前长度的最长子数组是否存在。

```go
func findLength(nums1 []int, nums2 []int) int {

	base := 131
	mod := 1000000007

	left, right := 0, min(len(nums1), len(nums2))
	var ans int
	for left <= right {
		mid := (left + right) / 2
		if check(nums1, nums2, mid, base, mod) {
			ans = mid
			left = mid + 1
		} else {
			right = mid - 1
		}
	}
	return ans
}

func check(nums1, nums2 []int, length, base, mod int) bool {
	hashSet := make(map[int]bool)

	hash := 0
	power := 1
	for i := 0; i < length; i++ {
		hash = (hash*base + nums1[i]) % mod
		if i < length-1 {
			power = (power * base) % mod
		}
	}
	hashSet[hash] = true

	for i := length; i < len(nums1); i++ {
		hash = ((hash-nums1[i-length]*power%mod+mod)%mod*base + nums1[i]) % mod
		hashSet[hash] = true
	}

	hash = 0
	for i := 0; i < length; i++ {
		hash = (hash*base + nums2[i]) % mod
	}
	if hashSet[hash] {
		return true
	}

	for i := length; i < len(nums2); i++ {
		hash = ((hash-nums2[i-length]*power%mod+mod)%mod*base + nums2[i]) % mod
		if hashSet[hash] {
			return true
		}
	}
	return false
}
```

