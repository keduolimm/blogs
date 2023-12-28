

[155. 最小栈](https://leetcode.cn/problems/min-stack/description)

栈的应用

维护两个栈

- 正常的栈
- 最小栈(单调栈)

这样模拟，即可

```python []
class MinStack:

    # 假设操作序列合法
    def __init__(self):
        self.stack = []
        self.minStack = [] # 维护单调栈

    def push(self, val: int) -> None:
        self.stack.append(val)
        if len(self.minStack) == 0 or self.minStack[-1] >= val:
            self.minStack.append(val)

    def pop(self) -> None:
        cur = self.stack.pop()
        if self.minStack[-1] == cur:
            self.minStack.pop()


    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.minStack[-1]

```