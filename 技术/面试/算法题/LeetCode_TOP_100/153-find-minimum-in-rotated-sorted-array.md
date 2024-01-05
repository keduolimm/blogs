

[153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/description)

边界超级容易错

- 特判单个，原本就是顺序的
- 然后二分，注意是>=

```c++ []
class Solution {
public:
    int findMin(vector<int>& nums) {
        int n = nums.size();
        // 特判
        if (nums[0] <= nums[n - 1]) {
            return nums[0];
        }

        int l = 0, r = n - 1;
        while (l <= r) {
            int m = l + (r - l) / 2;
            if (nums[m] >= nums[0]) {
                l = m + 1;
            } else {
                r = m - 1;
            }
        }
        return nums[l];
    }
};
```