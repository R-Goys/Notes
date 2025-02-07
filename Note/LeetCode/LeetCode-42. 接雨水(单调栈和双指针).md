[42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)



#  前言

记录一下接雨水这道题的我的思路过程，方便之后复习。

# 正文

题目很简单，

> 给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

## 双指针做法

首先介绍一下双指针的做法

初始状态，左指针指向数组第一个位置，右指针指向数组的最后一个位置，

同时利用一个变量pre储存左边界到左指针这一范围的最大值，一个变量re储存右边界到右指针这一范围的最大值，当当前pre值大于re值时，说明当前的右指针指向的位置如果小于变量re的值，那么它一定可以储存水，并且最大高度为re的值，与此同时就可以将这个位置的水的量加入答案之中，同时由于当前位置的值已经使用过，左移右指针

反之同理，如果当前re值小于pre值，说明当前左指针所指向位置的值若小于pre的值，那么这个位置一定可以储存水，并且储存水的最大绝对高度为pre，相对高度则为pre-当前左指针所指向位置的高度。

代码实现如下：

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int len = height.size();
        int i = 0, j = len - 1;
        int premax = 0, remax = 0;
        int ans = 0;
        while (i <= j) {
            premax = max(premax, height[i]);
            remax = max(remax, height[j]);
            if (premax >= remax)
                ans += remax - height[j--];//可以储存的水的量
            else
                ans += premax - height[i++];
        }
        return ans;
    }
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 栈做法

接下来介绍如何用单调栈来做这道题。

上面双指针的做法形象一点是一列一列地将雨水加入答案，而栈的做法则是一行一行的将储存的雨水加入答案：

首先，我们需要一个空的栈，然后从左到右依次将数组中的值加入栈中，

如果当前需要加入的值小于栈顶元素，那么继续加入即可，因为还没有遇到“桶”的右挡板，无法储存雨水，

而当前需要加入的值大于栈顶元素的时候，我们也不能心急，因为我们还需要判断桶的左挡板是否存在，这时候如何得到呢？先将当前栈顶元素top储存起来，然后弹出栈顶，随后的栈顶元素就是我们需要的左挡板，因为栈当中的元素是按从大到小排列的（接下来会说），此时如果栈为空，那么说明没有左挡板，如果栈中有数值，那么它一定比我们之前储存起来的栈top变量要大，此时，则说明我们top这个位置可以储存雨水，那么继续，我们可以通过当前遍历到的元素的下标减去当前栈顶（左挡板）的下标 - 1就可以得到我们的桶的宽度了，如何得到下标？很简单将下标存入栈就行了！

高则是通过min(左挡板的高度，右挡板的高度) - 桶底的高度（也就是储存起来的变量top）得到

此时就可以将（宽*高）加入我们的答案了，

此时还不能将遍历的指针后移！因为我们还不知道左侧是否还存在一个更高的挡板，之前我们也说了，我们使用栈是一行一行的将储存的雨水加入答案，因此我们需要将栈中的元素遍历，一直到栈顶元素大于当前遍历到数组元素的值，此时就算我们的这一个小“桶”加入完雨水了。

下面是代码的实现：

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int len = height.size();
        int ans = 0;
        stack<int> st;
        for (int i = 0; i < len; i++) {
            while (!st.empty() && height[st.top()] < height[i]) {
                int top = st.top();
                st.pop();
                if (st.empty())
                    break;
                int left = st.top();
                int width = i - left - 1;
                int hei = min(height[left], height[i]) - height[top];
                ans += hei * width;
            }
            st.push(i);
        }
        return ans;
    }
};
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 结语

这两种做法并非自己想出来的，而是通过看0x3f的视频学会的，然后通过自己的理解写出来的blog

目的是为了巩固自己的知识，希望也能帮助到你吧~

# -------END-------