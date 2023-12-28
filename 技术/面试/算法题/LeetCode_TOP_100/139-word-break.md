
[139. 单词拆分](https://leetcode.cn/problems/word-break/description)

思路: dp

可以借助Trie进行优化

```python []
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        n = len(s)
        dp = [False] * (n + 1)
        dp[0] = True
        for i in range(n):
            if not dp[i]:
                continue
            for w in wordDict:
                nw = len(w)
                if i + nw - 1 < n and s[i:i + nw] == w:
                    dp[i + nw] = True

        return dp[n]
                    
```