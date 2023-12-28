

[322. 零钱兑换](https://leetcode.cn/problems/coin-change/description)

完全背包

```python []
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        dp = [inf] * (amount + 1)
        dp[0] = 0
        for v in coins:
            for i in range(amount - v + 1):
                dp[i + v] = min(dp[i + v], dp[i] + 1)
        return -1 if dp[amount] == inf else dp[amount]
```