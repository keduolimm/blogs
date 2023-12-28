

[121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/description)

思路：贪心

```python []
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        res = 0
        pre = inf
        for v in prices:
            res = max(res, v - pre)
            pre = min(pre, v)

        return res
```