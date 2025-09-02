[36. 有效的数独](https://leetcode.cn/problems/valid-sudoku/)

> 请你判断一个 `9 x 9` 的数独是否有效。只需要 **根据以下规则** ，验证已经填入的数字是否有效即可。
>
> 1. 数字 `1-9` 在每一行只能出现一次。
> 2. 数字 `1-9` 在每一列只能出现一次。
> 3. 数字 `1-9` 在每一个以粗实线分隔的 `3x3` 宫内只能出现一次。（请参考示例图）
>
>  
>
> **注意：**
>
> - 一个有效的数独（部分已被填充）不一定是可解的。
> - 只需要根据以上规则，验证已经填入的数字是否有效即可。
> - 空白格用 `'.'` 表示。

---

除了暴力的做法，可以通过位运算快速搞定。

```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        vector<int> rows(9);
        vector<int> cols(9);
        vector<int> blocks(9);

        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] =='.') continue;
                int x = board[i][j] - '0';
                if (rows[i]>>x & 1 || cols[j]>>x & 1 ||
                blocks[(i/3)*3 + j/3]>>x & 1) {
                    return false;
                }

                rows[i] |= 1 << x;
                cols[j] |= 1 << x;
                blocks[(i/3)*3 + j/3] |= 1 << x;
            }
        }
        return true; 
    }
};
```



