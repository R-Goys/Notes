[239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)



#  前言

一个月前在acw上学习了队列，并做了滑动窗口这道题，一个月之后，我回到力扣，又遇见了这道题目，发现不会做了，深有感慨，故写此文，以便以后复习。

# 正文

## 题目分析

这道题想表达的意思就跟它的题目一样“滑动窗口”，然后在这个滑动的窗口之中，找到它的最大值，题目想要表达的意思很简单，但是怎么来做？

关于“滑动窗口”我们可以通过队列来进行表示，队头的元素表示当前窗口数值最大的下表，

而队尾解释起来就比较复杂了，队尾的元素所指向的值的意思可以用图片来表示![img](https://i-blog.csdnimg.cn/direct/d7401d74ddb24e74adf7c86c9426ed88.bmp)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

图片补充说明：尽管当前的加入窗口的值较小，但它在后续的窗口中可能会成为有用的参考。如果窗口内没有更大的值，它将一直存在于队列中，以备后续比较。

更新方式如下：

如果即将加入窗口的值比当前队尾的值更大，那么就应该移除队尾的值，如果这时候即将加入窗口的值还是比队尾的下表所指向的元素大，那么就继续移除队尾，直到队头遇见队尾，即队列变为空，这时候即将加入窗口的值，比去除队头的原窗口的任何一个索引所指向的值更大，于是，就可以将队尾（这时候队尾就是队头）更新为这个新加入窗口的的元素的下表的值，这个就是值当前更新后的当前窗口的最大值的索引，

具体的思想以上，那么接下来看看代码的实现：

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int sz = nums.size(); // 总长度
        vector<int> res;      // 用于存储结果
        int hh = 0, tt = -1;  // hh 表示队头，tt 表示队尾
        int val[100010];      // 存储元素索引的数组，固定大小足够容纳输入的大小

        // 遍历输入数组
        for(int i = 0; i < sz; i++)
        {
            // 如果当前最大值的索引超出窗口范围，就直接移除
            if(hh <= tt && i - val[hh] + 1 > k) hh++;

            // 移除队列中所有小于即将加入队列元素的索引
            while(hh <= tt && nums[i] >= nums[val[tt]]) tt--;

            // 将当前元素的索引添加到队尾
            val[++tt] = i;

            // 当当前索引大于等于窗口大小时，将当前窗口的最大值添加到结果中
            if(i + 1 >= k) res.push_back(nums[val[hh]]);
        }
        return res; // 返回结果
    }
};
```

# 结语

多复习才是正道，在往前学习的同时，也要记得回头看看，指不定哪天就忘了。

----

## 二刷

又忘了，唉唉，说一下思路加深记忆：

窗口由hh，tt两个变量维护，每次遍历将元素加入窗口，并且在窗口元素满的时候，应该去除窗口最左边的元素，如果要加入的值大于等于窗口最左边的元素，去除即可，这道题中的抽象比较多，所以不太好做，但是每次做加深理解就可以了，我认为有两个不好理解的地方：

1. window切片存储的真的就是滑动窗口中所有元素的下标吗？
2. hh和tt维护的到底是什么？

我觉得记住两点，hh始终是指向当前窗口的最大值，当前我们抽象出来的窗口始终是单调递减的。

有两种写法，不得不说切片真的好用

Cpp做法迁移过来的：

```go
func maxSlidingWindow(nums []int, k int) []int {
    n := len(nums)
    var ans []int
    window := make([]int, n)
    hh, tt := 0, -1
    for i := 0; i < n; i ++ {
        if hh <= tt && i - window[hh] + 1 > k {
            hh ++
        }
        for hh <= tt && nums[window[tt]] <= nums[i] {
            tt --
        }
        tt ++
        window[tt] = i
        if i - k + 1 >= 0 {
            ans = append(ans, nums[window[hh]])
        }
    }
    return ans
}
```

切片作为窗口：

```go
func maxSlidingWindow(nums []int, k int) []int {
    n := len(nums)
    var ans []int
    var window []int
    for i := 0; i < n; i ++ {
        if len(window) > 0 && i - window[0] >= k {
            window = window[1:]
        }
        for len(window) > 0 && nums[window[len(window) - 1]] <= nums[i] {
            window = window[:len(window) - 1]
        }
        window = append(window, i)
        if i - k + 1 >= 0 {
            ans = append(ans, nums[window[0]])
        }
    }
    return ans
}
```

