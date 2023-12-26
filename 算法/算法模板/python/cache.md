
---
# 前言

----

# 记忆化搜索

一般用于记忆化搜索的时候，可以直接使用
```python
@functools.cache
```

也可以使用
```python
@functools.lru_cache
```

当然如果程序爆MLE

再可能需要cache_clear一下，尤其在力扣是多个case一起连测的情况下

---

# 验证

力扣 62题

[不同路径](https://leetcode.cn/problems/unique-paths/description/)

```python []
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:

        @cache
        def dfs(r: int, c: int) -> int:
            if r == 0 or c == 0:
                return 1
            r1 = dfs(r - 1, c) if r > 0 else 0
            r2 = dfs(r, c - 1) if c > 0 else 0
            return r1 + r2
            
        res = dfs(m - 1, n - 1)
        dfs.cache_clear()

        return res
```


---
# 参考文献

[python中functools.cache用法详解及缓存策略问题](https://blog.csdn.net/weixin_44799217/article/details/129944149)

[如何使用Python内置缓存装饰器: @lru_cache，@cache 与 @cached_property](https://blog.csdn.net/captain5339/article/details/131434063)