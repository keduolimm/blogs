

[5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/description)

- DP

```python []
class Solution:
    def longestPalindrome(self, s: str) -> str:
        # 区间dp
        n = len(s)
        dp = [[0] * n for _ in range(n)]

        ans = s[0]
        for i in range(n):
            dp[i][i] = 1
        for w in range(2, n + 1):
            for i in range(n - w + 1):
                if w == 2:
                    if s[i] == s[i + w - 1]:
                        dp[i][i + w - 1] = 2
                        ans = s[i:i+w]
                else:
                    if s[i] == s[i + w - 1] and dp[i + 1][i + w - 2] > 0:
                        dp[i][i + w - 1] = dp[i + 1][i + w - 2] + 2
                        ans = s[i:i+w]

        return ans
```

- 中心扩展法

```python []
class Solution:
    def longestPalindrome(self, s: str) -> str:
        n = len(s)
        res = 0
        ans = ""
        # 中心扩展法
        for i in range(n):
            if 1 > res:
                res = 1
                ans = s[i]
            l, r = i - 1, i + 1
            while l >= 0 and r < n:
                if s[l] != s[r]:
                    break
                if r - l + 1 > res:
                    res = r - l + 1
                    ans = s[l:r+1]
                l -= 1
                r += 1

        for i in range(n - 1):
            if s[i] != s[i + 1]:
                continue
            if 2 > res:
                res = 2
                ans = s[i:i+2]

            l, r = i - 1, i + 2
            while l >= 0 and r < n:
                if s[l] != s[r]:
                    break
                if r - l + 1 > res:
                    res = r - l + 1
                    ans = s[l:r+1]
                l -= 1
                r += 1

        return ans
```