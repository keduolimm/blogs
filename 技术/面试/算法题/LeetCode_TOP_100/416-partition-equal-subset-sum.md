
[416. 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/description)

0-1 背包

经典问题模型转化

```python []
class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        # x1 + x2 + .. + y1 + y2 + ... = s
        # x1 + x2 .... = s // 2
        s = sum(nums)
        if s % 2 == 1:
            return False
        n = s // 2
        dp = [False] * (n + 1)
        dp[0] = True
        for v in nums:
            for j in range(n - v, -1, -1):
                if dp[j]:
                    dp[j + v] = True 
        return dp[n]
        
```