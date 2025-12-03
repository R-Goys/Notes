[210. 课程表 II](https://leetcode.cn/problems/course-schedule-ii/)

> 现在你总共有 `numCourses` 门课需要选，记为 `0` 到 `numCourses - 1`。给你一个数组 `prerequisites` ，其中 `prerequisites[i] = [ai, bi]` ，表示在选修课程 `ai` 前 **必须** 先选修 `bi` 。
>
> - 例如，想要学习课程 `0` ，你需要先完成课程 `1` ，我们用一个匹配来表示：`[0,1]` 。
>
> 返回你为了学完所有课程所安排的学习顺序。可能会有多个正确的顺序，你只要返回 **任意一种** 就可以了。如果不可能完成所有课程，返回 **一个空数组** 。

---

和课程表I一模一样，建立一个有向图，并且维护一个入度的数组，当入度为0的时候加入我们的广度搜索队列，每次遍历一个点，减少他能够到达的下一个点的入度。

```go
func findOrder(numCourses int, prerequisites [][]int) []int {
    cor := make([][]int, numCourses)
    in := make([]int, numCourses)
    q := make([]int, 0)
    ans := make([]int, 0)
    for _, pair := range prerequisites {
        in[pair[0]] ++
        cor[pair[1]] = append(cor[pair[1]], pair[0])
    }
    for i := 0; i < numCourses; i ++ {
        if in[i] == 0 {
            q = append(q, i)
        }
    }
    for len(q) > 0 {
        head := q[0]
        q = q[1:]
        ans = append(ans, head)
        for _, v := range cor[head] {
            in[v] --
            if in[v] == 0 {
                q = append(q, v)
            }
        }
    }
    if len(ans) != numCourses {
        return nil
    }
    return ans
}
```



