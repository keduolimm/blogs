

[22. 括号生成](https://leetcode.cn/problems/generate-parentheses/description)

dfs即可

0-1，取或者不取

卡特兰数，时间复杂度C(2n, n) / (n + 1)

$4^n / sqrt(n)$

```python []
class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        # 0-1, 保证前缀0的个数大于等于1

        res = []
        def dfs(n0: int, n1: int, s: List) -> None:
            if n0 > n or n1 > n:
                return
                
            if n0 == n and n1 == n:
                res.append(''.join(s))
                return

            s.append('(')
            dfs(n0 + 1, n1, s)
            s.pop()

            if n0 >= n1 + 1:
                s.append(')')
                dfs(n0, n1 + 1, s)
                s.pop()

        dfs(0, 0, [])
        return res
```