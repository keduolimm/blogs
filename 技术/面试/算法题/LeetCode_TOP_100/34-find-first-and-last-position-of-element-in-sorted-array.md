

[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/description)

```c++ []
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {

        vector<int> res;
        int l = 0, r = nums.size() - 1;
        while (l <= r) {
            int m = l + (r - l) / 2;
            if (nums[m] < target) {
                l = m + 1;
            } else {
                r = m - 1;
            }
        }
        if (l >= nums.size() || nums[l] != target) {
            return {-1, -1};
        }


        int left = l;
        int right = l;
        for (int i = left + 1; i < nums.size(); i++) {
            if (nums[i] == nums[left]) {
                right = i;
            } else {
                break;
            }
        }

        return {left, right};
    }
};
```