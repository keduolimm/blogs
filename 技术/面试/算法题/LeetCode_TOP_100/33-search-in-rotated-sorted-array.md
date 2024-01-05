

[33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/description)

思路:

1. 其实是可以找到分界点

如果不找分界点的话

那就需要二分的时候，分类讨论

还是原先的二分架构

然后在判断时，分情况讨论

2种情况下，在细分4种情况，一共8种情况，需要nums[0], 判定它在哪一个区间

经典好题

```c++ []
class Solution {
public:
    int search(vector<int>& nums, int target) {

        int n = nums.size();
        
        int l = 0, r = nums.size() - 1;
        while (l <= r) {
            int m = l + (r - l) / 2;
            
            if (nums[m] == target) return m;
            else if (nums[m] > target) {
                if (target >= nums[0] || (nums[m] < nums[0])) {
                    r = m - 1;
                } else {
                    l = m + 1;
                }
            } else {
                if (nums[m] >= nums[0] || (target < nums[0])) {
                    l = m + 1;
                } else {
                    r = m - 1;
                }
            }
        }

        return -1;

    }
};
```