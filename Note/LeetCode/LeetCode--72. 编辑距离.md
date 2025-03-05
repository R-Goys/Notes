[72. 编辑距离](https://leetcode.cn/problems/edit-distance/)



```go
func minDistance(word1 string, word2 string) int {
    n := len(word1)
    m := len(word2)
    f := make([][]int, 2)
    for i := 0; i < 2; i ++ {
        f[i] = make([]int, m + 1)
    }
    for i := 1; i <= m; i++ {
        f[0][i] = i
    }

    for i := 1; i <= n; i ++ {
        f[(i) % 2][0] = i
        for j := 1; j <= m; j ++ {
            if word1[i - 1] == word2[j - 1] {
                f[i % 2][j] = f[(i - 1) % 2][j - 1]
            } else {
                f[i % 2][j] = min(f[(i - 1) % 2][j], f[i % 2][j - 1], f[(i - 1) % 2][j - 1]) + 1
            }
        }
    }
    return f[n % 2][m]
}
```

需要知道之前三个状态来进行转移，直接利用滚动数组优化。

----

### 二刷

没有空间优化的版本：

```go
func minDistance(word1 string, word2 string) int {
    n := len(word1)
    m := len(word2)
    f := make([][]int, n + 1)
    for i := 0; i <= n; i ++ {
        f[i] = make([]int, m + 1)
    }
    
    for i := 1; i <= m; i++ {
        f[0][i] = i
    }
    for i := 1; i <= n; i++ {
        f[i][0] = i
    }

    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            if word1[i - 1] == word2[j - 1] {
                f[i][j] = f[i - 1][j - 1]
            } else {
                f[i][j] = min(f[i - 1][j], f[i][j - 1], f[i - 1][j - 1]) + 1
            }
        }
    }
    return f[n][m]
```



滚动数组优化：

值得注意的是，由于滚动数组我们难以直接在最开始就将j=0的情况全部初始化完，所以我们需要将他在DP状态转移的时候才能初始化。

```go
func minDistance(word1 string, word2 string) int {
    n := len(word1)
    m := len(word2)
    f := make([][]int, 2)
    for i := 0; i < 2; i ++ {
        f[i] = make([]int, m + 1)
    }
    
    for i := 1; i <= m; i++ {
        f[0][i] = i
    }

    for i := 1; i <= n; i ++ {
        f[(i) % 2][0] = i
        for j := 1; j <= m; j ++ {
            if word1[i - 1] == word2[j - 1] {
                f[i % 2][j] = f[(i - 1) % 2][j - 1]
            } else {
                f[i % 2][j] = min(f[(i - 1) % 2][j], f[i % 2][j - 1], f[(i - 1) % 2][j - 1]) + 1
            }
        }
    }
    return f[n % 2][m]
}
```

