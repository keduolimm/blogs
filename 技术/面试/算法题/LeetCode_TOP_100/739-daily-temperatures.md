

[739. 每日温度](https://leetcode.cn/problems/daily-temperatures/description)

单调栈的一个应用

逆向遍历，维护一个递减的单调栈

```python []
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        # 单调栈
        n = len(temperatures)
        res = [0] * n
        monostack = []
        for i in range(len(temperatures) - 1, -1, -1):
            while len(monostack) > 0 and temperatures[monostack[-1]] <= temperatures[i]:
                monostack.pop()
            if len(monostack) == 0:
                pass
            else:
                res[i] = monostack[-1] - i
            monostack.append(i)
        
        return res
```