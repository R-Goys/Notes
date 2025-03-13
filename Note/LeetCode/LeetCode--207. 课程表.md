[207. 课程表](https://leetcode.cn/problems/course-schedule/)

> 你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。
>
> 在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。
>
> - 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。
>
> 请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

---

经典拓扑，先将入度和边都计算出来，然后将入度为0(不需要先修课程)的课加入队列，开始广搜，每走一条边，降低对象的入度，如果为0，就加入队列，不为零就不管。

```go
func canFinish(numCourses int, prerequisites [][]int) bool {
    in := make([]int, numCourses)
    edge := make([][]int, numCourses)
    ans := make([]int, 0)
    n := len(prerequisites)
    for i := 0; i < n; i ++ {
        in[prerequisites[i][0]]++
        edge[prerequisites[i][1]] = append(edge[prerequisites[i][1]], prerequisites[i][0])
    }
    q := []int{}
    for i := 0; i < numCourses; i ++ {
        if in[i] == 0 {
            q = append(q, i)
        }
    } 

    for len(q) > 0 {
        u := q[0]
        q = q[1:]
        ans = append(ans, u)
        for _, v := range edge[u] {
            in[v]--
            if in[v] == 0 {
                q = append(q, v)
            }
        }
    }
    return len(ans) == numCourses
}
```

