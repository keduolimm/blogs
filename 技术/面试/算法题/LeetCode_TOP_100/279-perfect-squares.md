

[279. 完全平方数](https://leetcode.cn/problems/perfect-squares/description)

完全背包

```python []
class Solution:
    def numSquares(self, n: int) -> int:
        # 完全背包
        dp = [inf] * (n + 1)
        dp[0] = 0
        m = int(sqrt(n))
        print (n, m)
        for i in range(1, m + 1):
            for j in range(0, n - i * i + 1):
                dp[j + i * i] = min(dp[j + i * i], dp[j] + 1)
        return dp[n]
```