[406. 根据身高重建队列](https://leetcode.cn/problems/queue-reconstruction-by-height/)

> 假设有打乱顺序的一群人站成一个队列，数组 `people` 表示队列中一些人的属性（不一定按顺序）。每个 `people[i] = [hi, ki]` 表示第 `i` 个人的身高为 `hi` ，前面 **正好** 有 `ki` 个身高大于或等于 `hi` 的人。
>
> 请你重新构造并返回输入数组 `people` 所表示的队列。返回的队列应该格式化为数组 `queue` ，其中 `queue[j] = [hj, kj]` 是队列中第 `j` 个人的属性（`queue[0]` 是排在队列前面的人）。

----

先排序，此时按身高从小到大排序，相同时，排在前面的人的数量更大的排在前面，然后我们可以从小到大遍历，按大小将每个人放到指定位置，排在他前面的身高更高的人数为 person[1]，并且这些人一定是后放入队列的，那么我们可以知道，我们必须要跳过 person[1] 个空格，才能找到正确的位置，然后放入队列，而后来的人的身高必定大于之前放入队列的，而为空的格子，说明还没有放入队列，而且这个人必定大于先放入队列的人的身高，所以可以放心。

```cpp
class Solution {
public:
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(), people.end(), [](const vector<int>& u, const vector<int>& v) {
            return u[0] < v[0] || (u[0] == v[0] && u[1] > v[1]);
        });
        int n = people.size();
        vector<vector<int>> ans(n);

        for (const vector<int>& person : people) {
            int spaces = person[1] + 1;
            for (int i = 0; i < n; i ++) {
                if (ans[i].empty()) {
                    spaces --;
                    if (!spaces) {
                        ans[i] = person;
                        break;
                    }
                }
            }
        }
        return ans;
    }
};
```

方法二，先插入大的，小的随便插都没事

```cpp
class Solution {
public:
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(), people.end(), [](const vector<int>& u, const vector<int>& v) {
            return u[0] > v[0] || (u[0] == v[0] && u[1] < v[1]);
        });
        int n = people.size();
        vector<vector<int>> ans;
        for (const vector<int> person : people) {
            ans.insert(ans.begin() + person[1], person);
        }
        return ans;
    }
};
```

