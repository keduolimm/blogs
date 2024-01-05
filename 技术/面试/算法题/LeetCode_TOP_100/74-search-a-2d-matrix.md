
[74. 搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/description)

```c++ []
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        // 把二维看成一维，即可
        int h = matrix.size(), w = matrix[0].size();
        int l = 0, r = h * w - 1;
        while (l <= r) {
            int m = l + (r - l) / 2;
            int y = m / w, x = m % w;
            if (matrix[y][x] == target) return true;
            else if (matrix[y][x] > target) {
                r = m - 1;
            } else {
                l = m + 1;
            }
        }
        return false;
    }
};
```