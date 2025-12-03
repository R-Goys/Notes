[79. 单词搜索](https://leetcode.cn/problems/word-search/)

> 给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。
>
> 单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

----

dfs，虽然这里我直接修改board了，但是最好还是额外用一个st二维数组来表示是否遍历。

```go
func exist(board [][]byte, word string) bool {
    for i := 0; i < len(board); i ++ {
        for j := 0; j < len(board[0]); j ++ {
            if dfs(i, j, 0, len(word), board, word) {
                return true
            }
        }
    }
    return false
}

func dfs(i, j, depth, n int, board [][]byte, word string) bool {
    if board[i][j] != word[depth] {
        return false
    }
    if depth == n - 1 {
        return true
    }
    dx := []int{0, 0, -1, 1}
    dy := []int{-1, 1, 0, 0}
    board[i][j] = 0
    for k := 0; k < 4; k ++ {
        x := i + dx[k]
        y := j + dy[k]
        if x >= 0 && x < len(board) && y >= 0 && y < len(board[0]) && dfs(x ,y , depth + 1, n , board, word) {
            return true
        }
    }
    board[i][j] = word[depth]
    return false
}
```

