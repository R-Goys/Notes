[1301. 最大得分的路径数目](https://leetcode.cn/problems/number-of-paths-with-max-score/)

> 给你一个正方形字符数组 `board` ，你从数组最右下方的字符 `'S'` 出发。
>
> 你的目标是到达数组最左上角的字符 `'E'` ，数组剩余的部分为数字字符 `1, 2, ..., 9` 或者障碍 `'X'`。在每一步移动中，你可以向上、向左或者左上方移动，可以移动的前提是到达的格子没有障碍。
>
> 一条路径的 「得分」 定义为：路径上所有数字的和。
>
> 请你返回一个列表，包含两个整数：第一个整数是 「得分」 的最大值，第二个整数是得到最大得分的方案数，请把结果对 **`10^9 + 7`** **取余**。
>
> 如果没有任何路径可以到达终点，请返回 `[0, 0]` 。

---

最答辩，看了半天结果还能斜着移动，不想再写第二遍的题

```go
func pathsWithMaxScore(board []string) []int {
    n := len(board)
    mod := 1000000007
    board[n - 1] = board[n - 1][:n - 1] + "0"
    
    dp := make([][]int, n + 1)
    cnt := make([][]int, n + 1)
    
    for i := 0; i <= n; i ++ {
        dp[i] = make([]int, n + 1)
        cnt[i] = make([]int, n + 1)
    }
    
    cnt[1][1] = 1
    
    for i := 1; i <= n; i ++ {
        for j := 1; j <= n; j ++ {
            if board[i - 1][j - 1] != 'X' && (cnt[i - 1][j - 1] > 0 || cnt[i - 1][j] > 0 || cnt[i][j - 1] > 0) {
                mx := max(dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1])
                dp[i][j] = (mx + int(board[i - 1][j - 1]-'0')) % mod
                
                if dp[i - 1][j - 1] == mx {
                    cnt[i][j] = (cnt[i][j] + cnt[i - 1][j - 1]) % mod
                }
                if dp[i][j - 1] == mx {
                    cnt[i][j] = (cnt[i][j] + cnt[i][j - 1]) % mod
                }
                if dp[i - 1][j] == mx {
                    cnt[i][j] = (cnt[i][j] + cnt[i - 1][j]) % mod
                }
            }
        }
    }
    
    return []int{dp[n][n], cnt[n][n]}
}
```

