[1691. 堆叠长方体的最大高度](https://leetcode.cn/problems/maximum-height-by-stacking-cuboids/)

> 给你 `n` 个长方体 `cuboids` ，其中第 `i` 个长方体的长宽高表示为 `cuboids[i] = [widthi, lengthi, heighti]`（**下标从 0 开始**）。请你从 `cuboids` 选出一个 **子集** ，并将它们堆叠起来。
>
> 如果 `widthi <= widthj` 且 `lengthi <= lengthj` 且 `heighti <= heightj` ，你就可以将长方体 `i` 堆叠在长方体 `j` 上。你可以通过旋转把长方体的长宽高重新排列，以将它放在另一个长方体上。
>
> 返回 **堆叠长方体** `cuboids` 可以得到的 **最大高度** 。

---

对于这个三维的物体，首先对其长宽高按升序排序，这样可以保证旋转得到最优解。（这里不太会证明）

然后将所有物体按照长宽高进行升序排序，以此来保证我们可以正确进行状态转移，然后直接进行 dp 即可。

为什么不排序就无法进行正确的状态转移？比方说，对于很多长方体的集合，当我们 dp 到其中一个大小适中，本应该放在已经计算出来的最大值的中间，但是我们如何判断他是否合适加入到中间的一层状态，又如何去更新其他的状态？此时他难以被计算进结果，因为此时对于他的长和宽的判断变得棘手，我们可以类比俄罗斯信封那道题，但是如果排序了之后，我们可以恰好地先计算最顶层的方块，然后依次对更大的长方体进行 dp 。

```go
func maxHeight(cuboids [][]int) int {
    for _, c := range cuboids {
        sort.Ints(c)
    }
    ans := 0
    n := len(cuboids)
    sort.Slice(cuboids, func(i, j int) bool {
        return cuboids[i][0] < cuboids[j][0] || 
            (cuboids[i][0] == cuboids[j][0] && cuboids[i][1] < cuboids[j][1]) ||
            (cuboids[i][0] == cuboids[j][0] && cuboids[i][1] == cuboids[j][1] && cuboids[i][2] < cuboids[j][2])
    })
    f := make([]int, n)
    for i, c2 := range cuboids {
        for j, c1 := range cuboids[:i] {
            if c1[1] <= c2[1] && c1[2] <= c2[2] {
                f[i] = max(f[i], f[j])
            }
        }
        f[i] += c2[2]
        ans = max(ans, f[i])
    }
    return ans
}
```

