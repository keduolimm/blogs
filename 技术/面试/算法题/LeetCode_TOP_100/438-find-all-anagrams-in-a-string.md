

[438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/description)


```python []
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        n, m = len(s), len(p)
        if m > n:
            return []
        
        # 特征压缩
        h1 = Counter(p)
        h2 = Counter(s[0:m - 1])

        res = []
        for i in range(m - 1, n):
            h2[s[i]] += 1
            if h1 == h2:
                res.append(i + 1 - m)
            h2[s[i + 1 - m]] -= 1
        
        return res
```

python的Counter真的太好用了




