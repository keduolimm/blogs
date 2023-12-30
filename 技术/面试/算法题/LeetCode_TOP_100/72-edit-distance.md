

[72. 编辑距离](https://leetcode.cn/problems/edit-distance/description)


```python []
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:

        n, m = len(word1), len(word2)

        if len(word2) == 0:
            return n
        if len(word1) == 0:
            return m

        dp = [[inf] * m for _ in range(n)]

        dp[0][0] = 0 if word1[0] == word2[0] else 1
        for i in range(1, m):
            dp[0][i] = dp[0][i - 1] + 1
            if word2[i] == word1[0]:
                dp[0][i] = min(dp[0][i], i)

        for i in range(1, n):
            dp[i][0] = dp[i - 1][0] + 1
            if word1[i] == word2[0]:
                dp[i][0] = min(dp[i][0], i)


        for i in range(n):
            for j in range(m):
                if i == 0 or j == 0:
                    continue
                else:
                    if word1[i] == word2[j]:
                        dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1])
                    else:
                        dp[i][j] = min(dp[i][j - 1], dp[i - 1][j], dp[i - 1][j - 1]) + 1
            
        return dp[n - 1][m - 1]
                
```