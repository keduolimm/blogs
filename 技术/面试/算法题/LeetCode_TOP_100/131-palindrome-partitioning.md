
[131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/description)

思路: 回溯

```python []
class Solution:
    def partition(self, s: str) -> List[List[str]]:
        # # 刷表法
        # n = len(s)
        # dp = [0] * (n + 1)
        # dp[0] = 1
        # for i in range(n):
        #     if dp[i] == 0:
        #         continue
        #     for j in range(i, n):
        #         if s[i:j+1] == s[i:j+1][::-1]:
        #             dp[j+1] += dp[i]
        # return dp[n]

        n = len(s)
        res = []
        def dfs(i: int, arr: List) -> None:
            if i >= n:
                res.append(arr[:])
                return
            for j in range(i, n):
                if s[i:j+1] == s[i:j+1][::-1]:
                    arr.append(s[i:j+1])
                    dfs(j+1, arr)
                    arr.pop()

        dfs(0, [])
        return res
```