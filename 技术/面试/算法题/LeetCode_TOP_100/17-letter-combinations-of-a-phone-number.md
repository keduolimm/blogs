

[17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/description)

思路: 回溯

```python
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        hp = {}
        hp['2'] = "abc"
        hp['3'] = "def"
        hp['4'] = "ghi"
        hp['5'] = "jkl"
        hp['6'] = "mno"
        hp['7'] = "pqrs"
        hp['8'] = "tuv"
        hp['9'] = "wxyz"

        res = []
        def dfs(s: int, arr: []):
            if s >= len(digits):
                res.append(''.join(arr))
                return
            for c in hp[digits[s]]:
                arr.append(c)
                dfs(s + 1, arr)
                arr.pop()
        if len(digits) > 0:
            dfs(0, [])
        return res
```