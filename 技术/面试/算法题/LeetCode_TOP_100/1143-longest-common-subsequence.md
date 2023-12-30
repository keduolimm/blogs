

[1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/description)

```python []
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        n, m = len(text1), len(text2)
        dp = [[0] * m for _ in range(n)]

        for i in range(n):
            for j in range(m):
                if text1[i] == text2[j]:
                    dp[i][j] = 1
                    if i > 0 and j > 0:
                        dp[i][j] += dp[i - 1][j - 1]
                else:
                    dp[i][j] = 0
                    if i > 0:
                        dp[i][j] = max(dp[i][j], dp[i - 1][j])
                    if j > 0:
                        dp[i][j] = max(dp[i][j], dp[i][j - 1])
        
        return dp[n - 1][m - 1]
```