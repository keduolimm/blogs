

[763. 划分字母区间](https://leetcode.cn/problems/partition-labels/description/)

区间分割 + 区间合并(借助栈来实现)

```python []
class Solution:
    def partitionLabels(self, s: str) -> List[int]:
        pre = Counter()
        suf = Counter()
        for i in range(len(s)):
            c = s[i]
            if pre[c] == 0:
                pre[c] = i + 1
                suf[c] = i + 1
            else:
                suf[c] = i + 1
        
        arr = []
        for k, v in pre.items():
            arr.append((v, suf[k]))

        arr.sort(key = lambda x: x[0])

        stack = []
        for e in arr:
            s1, e1 = e[0], e[1]
            while len(stack) > 0 and stack[-1][1] >= s1:
                cur = stack.pop()
                s1 = min(s1, cur[0])
                e1 = max(e1, cur[1])
            stack.append((s1, e1))
        
        return [e[1] - e[0] + 1 for e in stack]
```