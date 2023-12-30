
[62. 不同路径](https://leetcode.cn/problems/unique-paths/description)

```python []
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:

        @cache
        def dfs(r: int, c: int) -> int:
            if r == 0 or c == 0:
                return 1
            r1 = dfs(r - 1, c) if r > 0 else 0
            r2 = dfs(r, c - 1) if c > 0 else 0
            return r1 + r2
            
        res = dfs(m - 1, n - 1)
        dfs.cache_clear()

        return res
```