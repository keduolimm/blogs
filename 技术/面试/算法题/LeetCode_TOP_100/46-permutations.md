

[46. 全排列](https://leetcode.cn/problems/permutations/description)

思路: 回溯

```python []
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        res = []
        def dfs(vis: List[bool], trace: List[int]):
            if len(trace) == len(nums):
                res.append(trace[:])
                return
            for i in range(len(nums)):
                if not vis[i]:
                    vis[i] = True
                    trace.append(nums[i])
                    dfs(vis, trace)
                    trace.pop()
                    vis[i] = False
        
        dfs([False] * len(nums), [])
        return res
```