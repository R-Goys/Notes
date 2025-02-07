[529. 扫雷游戏](https://leetcode.cn/problems/minesweeper/)

#  题目分析

![img](https://i-blog.csdnimg.cn/direct/f3babb5a5b4847d3972163663b877ca3.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

初步分析，我们可以知道这个扫雷大体上和我们玩的经典扫雷并没有什么不同，
 题目可以翻译为：选择一个坐标来点击，如果点到了雷’M‘，那就将这个坐标上的字符修改为’X‘

如果点到了没有相邻地雷的点，那就会将这个点周围八个点自动点开，然后如果这八个点又存在没有相邻地雷的点，那就再将它周围的八个点自动点开，一旦遇到地雷相邻的点后，就将这个点周围的雷统计出来（这一步很简单，因为我们预先就知道了地雷’M’在哪里），然后将地雷的数量赋给这个与地雷相邻的点，但是与此同时就不再继续点开这个点周围的八个点了。



由于每次点开“周围没有相邻地雷的点”的时候，我们都需要再遍历周围的八个点来重复一遍这个“点开一个点”的操作，我们可以联想到利用队列来储存，进一步想到通过广搜来解决这个问题。

# 代码实现

于是思路就可以化成代码：

```cpp
class Solution {
public:
    vector<vector<char>> updateBoard(vector<vector<char>>& board, vector<int>& click) {
        int r = click[0], c = click[1]; // 获取当前点击的位置
        return bfs(r, c, board); // 调用广搜函数处理点击事件
    }
    
private:
    // 广搜函数
    vector<vector<char>> bfs(int r, int c, vector<vector<char>>& board) {
        int st[100][100]; // 状态数组，跟踪已访问的单元格
        
        // 如果点击的是雷，直接标记为 'X' 并返回
        if (board[r][c] == 'M') {
            board[r][c] = 'X';
            return board;
            //此时游戏结束
        }
        
        // 如果点击的是空单元，开始 BFS
        if (board[r][c] == 'E') {
            pair<int, int> q[5000]; // 坐标的储存使用pair
            int hh = 0, tt = -1; // 队列头尾指针
            int xx[8] = {1, 0, -1, 0, 1, 1, -1, -1}; // 当前点周围的行偏移量
            int yy[8] = {0, 1, 0, -1, 1, -1, -1, 1}; // 当前点周围的列偏移量
            
            // 将起始点击位置加入队列
            q[++tt] = {r, c};
            st[r][c] = 1; // 标记为已访问
            
            // 当队列不为空时继续处理
            while (hh <= tt) {
                int count = 0; // 当前单元周围雷的数量
                r = q[hh].first; // 获取当前行索引
                c = q[hh++].second; // 获取当前列索引，并移动队列头
                st[r][c] = 0; // 重置状态标记
                
                // 检查周围的 8 个方向
                for (int i = 0; i < 8; i++) {
                    int r1 = r, c1 = c; // 存储当前单元周围的坐标
                    r1 += xx[i]; // 行偏移
                    c1 += yy[i]; // 列偏移
                    
                    // 确保索引在合法范围内
                    if (r1 < 0 || r1 >= board.size() || c1 < 0 || c1 >= board[0].size()) continue;
                    // 计算当前单元周围的雷的数量
                    if (board[r1][c1] == 'M') count++;
                }
                
                // 根据周围雷的数量更新当前单元格
                if (count == 0) board[r][c] = 'B'; // 如果周围没有雷，标记为 'B'
                if (count > 0) board[r][c] = count + '0'; // 将雷的数量转换为字符并更新
                
                // 如果周围没有雷，则继续检查新的单元格
                // 有人可能会问，为什么不能将两个for循环放在一起
                // 我最开始也是这样想的，但是实际上会出问题
                // 原因就是当我们还没有查找完周围雷的数量的时候，我们不能断定周围就没有雷！
                // 因此要分两步，先查找雷，再判断这个点是否为“周围没有雷的格子”
                for (int i = 0; i < 8; i++) {
                    int r1 = r, c1 = c; // 存储当前单元周围的坐标
                    r1 += xx[i]; // 通过xx和yy数组来遍历周围的八个点
                    c1 += yy[i];
                    
                    // 确保索引在合法范围内
                    if (r1 < 0 || r1 >= board.size() || c1 < 0 || c1 >= board[0].size()) continue;
                    // 如果当前单元是空单元且未访问过且周围没有雷
                    if (board[r1][c1] == 'E' && st[r1][c1] == 0 && count == 0) {
                        q[++tt] = {r1, c1}; // 将新单元加入队列
                        st[r1][c1] = 1; // 标记为已访问
                    }
                }
            }
        }
        
        // 返回board
        return board;
    }
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 结语

总的来说，这道题是一道简单的广搜模板题，多做题，多思考，学会判断题目在考我们什么，只要知道了这一点，就很简单了，剩下的就是调bug了（逃）

# ————END————