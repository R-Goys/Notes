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