

函数闭包

```c++ []
function<int(int)> dfs = [&](int p) {
    if (p <= 0) return 0;
    return p + dfs(p - 1);
};
```

