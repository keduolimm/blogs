

[32. 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/description)

思路: 栈

区间合并，区间相邻合并

```python []
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        intervals = []
        stack = []
        for i in range(len(s)):
            if s[i] == '(':
                stack.append(i)
            else:
                if len(stack) > 0:
                    l = stack.pop()
                    intervals.append([l, i])

        intervals.sort(key=lambda x: x[0])
        
        res = 0
        j = 0
        while j < len(intervals):
            e = intervals[j]
            k = j + 1
            while k < len(intervals) and intervals[k][0] <= e[1] + 1:
                e[1] = max(e[1], intervals[k][1])
                k += 1
            res = max(res, e[1] - e[0] + 1)
            j = k
        
        return res
```