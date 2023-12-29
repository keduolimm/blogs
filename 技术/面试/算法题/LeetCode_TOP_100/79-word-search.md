
[79. 单词搜索](https://leetcode.cn/problems/word-search/description)

dfs编写

出口，递归

技巧
```python
pairwise([1, 0, -1, 0, 1])
```

# 代码

```python []
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        h, w = len(board), len(board[0])
        keycode = lambda y, x: y * w + x

        m = len(word)
        def dfs(s: int, y: int, x: int, hash: set) -> bool:
            if board[y][x] != word[s]:
                return False
            hash.add(keycode(y, x))
            if s + 1 >= m:
                return True

            for dy, dx in pairwise([1, 0, -1, 0, 1]):
                ty, tx = y + dy, x + dx
                if 0 <= ty < h and 0 <= tx < w:
                    k = keycode(ty, tx)
                    if k not in hash:
                        if dfs(s + 1, ty, tx, hash):
                            return True

            hash.remove(keycode(y, x))
            return False

        for i in range(h):
            for j in range(w):
                if dfs(0, i, j, set()):
                    return True
        return False
```