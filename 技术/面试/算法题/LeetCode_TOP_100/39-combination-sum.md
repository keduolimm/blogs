
[39. 组合总和](https://leetcode.cn/problems/combination-sum/description)

思路: 搜索/DFS

```python []
class Solution:
    def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
        res = []

        n = len(candidates)
        def dfs(s: int, acc: int, arr: List[int]) -> None:
            if acc == target:
                res.append(arr[:])
                return
            if s >= n or acc > target:
                return

            for i in range(target + 1):
                if i * candidates[s] + acc > target:
                    break
                tmp = [candidates[s]] * i
                arr1 = arr + tmp
                dfs(s + 1, i * candidates[s] + acc, arr1)
        
        dfs(0, 0, [])
        return res
            
```