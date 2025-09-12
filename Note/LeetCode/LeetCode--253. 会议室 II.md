[253. 会议室 II](https://leetcode.cn/problems/meeting-rooms-ii/)

> 给你一个会议时间安排的数组 `intervals` ，每个会议时间都会包括开始和结束的时间 `intervals[i] = [starti, endi]` ，返回 *所需会议室的最小数量* 。

---

直接排序，双指针，移动的指针表示当前时间线，如果当前时间线大于了之前召开会议的结束的时间，就可以回收，反之，每次移动最新的时间线都要新增一个房间，换句话说，每次移动指针类比为公交车经过站，且每次只会有一个人上车，所以每次经过站，需要的座位都 ++，如果经过这个站有人要下车，无论下多少个都不会影响需要的最大座位，所以--。

```cpp
class Solution {
public:
    int minMeetingRooms(vector<vector<int>>& intervals) {
        vector<int> start;
        vector<int> end;
        int room = 0;

        for (auto interval : intervals) {
            start.push_back(interval[0]);
            end.push_back(interval[1]);
        }

        sort(start.begin(), start.end());
        sort(end.begin(), end.end());

        int endPointer = 0, startPointer = 0;

        while(startPointer < intervals.size()) {
            if (start[startPointer] >= end[endPointer]) {
                room --;
                endPointer ++;
            }
            room ++;
            startPointer ++;
        }
        return room;
    }
};  
```

另一个思路是堆排序，先给所有会议排序，然后遍历，每次将会议的结束时间塞入堆中，遍历过程中发现堆顶的结束时间大于当前遍历的元素的起始时间

```cpp
class Solution {
public:
    int minMeetingRooms(vector<vector<int>>& intervals) {
        if (intervals.size() == 0) {
            return 0;
        }
        priority_queue<int, vector<int>, greater<int>> allocator;

        sort(intervals.begin(), intervals.end());

        allocator.emplace(intervals[0][1]);

        for (int i = 1; i < intervals.size(); i ++) {
            if (intervals[i][0] >= allocator.top()) {
                allocator.pop();
            }

            allocator.emplace(intervals[i][1]);
        }

        return allocator.size();
    }
};
```

