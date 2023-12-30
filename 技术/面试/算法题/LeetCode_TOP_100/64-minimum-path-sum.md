

[64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/description)


```python []
class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:
        h, w = len(grid), len(grid[0])
        dp = [[inf] * w for _ in range(h)]
        for i in range(h):
            for j in range(w):
                if i == 0 and j == 0:
                    dp[i][j] = grid[i][j]
                elif i == 0:
                    dp[i][j] = dp[i][j - 1] + grid[i][j]
                elif j == 0:
                    dp[i][j] = dp[i - 1][j] + grid[i][j]
                else:
                    dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j]
        return dp[h - 1][w - 1]
```