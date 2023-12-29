

[51. N 皇后](https://leetcode.cn/problems/n-queens/description)

经典的回溯题

```python []
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        #rows, cols = 0, 0

        res = []
        def solve(arr: List) -> List[str]:
            board = [['.'] * n for i in range(n)]
            for i in range(len(arr)):
                board[i][arr[i]] = 'Q'
            return [''.join(x) for x in board]

        def dfs(r: int, s1: int, s2: int, s3: int, arr: List):
            if r >= n:
                res.append(solve(arr))
                return
            for i in range(n):
                if ((1 << i) & s1) == 0 \
                and ((1 << (r + i)) & s2) == 0 \
                and ((1 << (r - i + n)) & s3) == 0:
                    arr.append(i)
                    dfs(r + 1, s1 | (1 << i), (1 << (r + i)) | s2, (1 << (r - i + n)) | s3, arr)
                    arr.pop()
        
        dfs(0, 0, 0, 0, [])
        return res
```