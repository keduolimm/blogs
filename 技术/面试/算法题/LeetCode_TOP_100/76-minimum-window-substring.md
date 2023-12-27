


[76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/description)

```python []
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        cnt = Counter(t)

        j = 0
        scnt = Counter()

        def comp() -> bool:
            for k, v in cnt.items():
                if scnt[k] < v:
                    return False
            return True

        res = ""
        for i in range(len(s)):
            scnt[s[i]] += 1
            while j <= i and comp():
                scnt[s[j]] -= 1
                j += 1
            if j > 0:
                if res == "" or len(res) > len(s[j-1:i+1]):
                    res = s[j-1:i+1]

        return res
```

也属于滑窗类型

