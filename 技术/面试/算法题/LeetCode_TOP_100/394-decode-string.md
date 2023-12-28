

[394. 字符串解码](https://leetcode.cn/problems/decode-string/description)

这题写起来有点绕

好像一个栈就行了，还真的不需要借助递归函数

```python []
class Solution:
    def decodeString(self, s: str) -> str:

        def dfs(l: int, r: int) -> str:
            stack = []
            tmp = 0
            for i in range(l, r):
                c = s[i]
                if c >= '0' and c <= '9':
                    tmp = tmp * 10 + (ord(c) - ord('0'))
                elif c == '[':
                    stack.append((0, tmp))
                    stack.append((1, '['))
                    tmp = 0
                elif c == ']':
                    s1 = ""
                    while stack[-1][0] != 1:
                        s1 += (stack[-1][1][::-1])
                        stack.pop()
                    stack.pop() # '['
                    _, cnt = stack.pop() # 数
                    s1 = s1[::-1]
                    stack.append((2, s1 * (int)(cnt)))
                else:
                    stack.append((2, c))

            s2 = ""
            for cur in stack:
                s2 += (cur[1])

            return s2

        return dfs(0, len(s))
```
