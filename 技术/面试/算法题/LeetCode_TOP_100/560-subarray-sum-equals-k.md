

[560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/submissions/491353514)

```python []
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        res = 0
        mp = Counter()
        mp[0] = 1

        acc = 0
        for i in range(len(nums)):
            acc += nums[i]
            res += mp[acc - k]
            mp[acc] += 1
        return res

```