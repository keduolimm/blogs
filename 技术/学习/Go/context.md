
context 支持链式调用

它构建一个树

父子协程之间，交互通讯的一种方式

emptyCtx

```Context
Done() <- chan struct{}  // 只读
Err error // 只有在cancel和timeout的情况下，才会存在
Value()
```

context -> cancelCtx -> timeoutCtx

timeoutCtx 包含 Cancel和定时器

在调用链中，父Context能影响到子Context

而value的查找，是逆序往上的


