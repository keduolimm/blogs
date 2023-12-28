

[20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/description)

栈的一个应用

```python []
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []
        for c in s:
            if c in '({[':
                stack.append(c)
            else:
                if len(stack) == 0:
                    return False
                top = stack.pop()
                if c == ')' and top != '(':
                    return False
                elif c == '}' and top != '{':
                    return False
                elif c == ']' and top != '[':
                    return False
        return len(stack) == 0                  
```