

[3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/description/)


```python []
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        res = 0
        j = 0
        hash = Counter()
        for i in range(len(s)):
            p = ord(s[i])
            hash[p] += 1
            while j <= i and hash[p] >= 2:
                p1 = ord(s[j])
                hash[p1] -= 1
                j += 1
            res = max(res, i - j + 1)

        return res
```

Counter的使用

滑窗的应用

