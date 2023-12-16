
# 前言

树状数组大致分为两类

- 普通树状数组
- 支持RMQ操作的树状数组


# 普通的树状数组

特点
- 点修改
- 区间查询

功能点

- 支持kth查询

```java
public class BIT {

    int n;
    int[] arr;
    int level = 0;

    public BIT(int n) {
        this.n = n;
        this.arr = new int[n + 1];
        this.level = (int)(Math.log(n) / Math.log(2)) + 1;
    }

    void update(int p, int d) {
        while (p <= n) {
            arr[p] += d;
            p += p &-p;
        }
    }

    int query(int p) {
        int res = 0;
        while (p > 0) {
            res += arr[p];
            p -= p & -p;
        }
        return res;
    }

    // log(n)支持寻找kth元素
    int kth(int k) {
        int res = 0;
        for (int i = level; i >= 0; i--) {
            int bin = 1 << i;
            if (res + bin <= n && k > arr[res + bin]) {
                k -= arr[res + bin];
                res += bin;
            }
        }
        return res + 1;
    }
    
}
```

---
# 支持RMQ的树状数组

```java
public class RMQBIT<T> {
    int n;
    Object[] ori;
    Object[] arr;
    Supplier<T> e;
    BiFunction<T, T, T> op;

    public RMQBIT(int n, Supplier<T> e, BiFunction<T, T, T> op) {
        this.n = n;
        this.ori = new Object[n + 1];
        this.arr = new Object[n + 1];
        this.e = e;
        this.op = op;

        for (int i = 0; i <= n; i++) {
            this.ori[i] = e.get();
        }
        for (int i = 0; i <= n; i++) {
            this.arr[i] = e.get();
        }
    }

    public void setAll(T[] ori) {
        for (int i = 1; i <= n; i++) {
            this.ori[i] = this.arr[i] = ori[i];
            for (int j = 1; j < lowbit(i); j <<= 1) {
                arr[j] = this.op.apply((T)arr[j], (T)arr[i - j]);
            }
        }
    }

    // O(log(n)), 前缀查询
    public T query(int p) {
        T res = (T)arr[p];
        while (p > 0) {
            res = this.op.apply(res, (T)arr[p]);
            p -= lowbit(p);
        }
        return res;
    }

    // 范围查询，操作时间复杂度, O(log(n)^2)
    public T query(int l, int r) {
        T ans = (T)ori[r];
        while (r >= l) {
            ans = this.op.apply(ans, (T)ori[r]);
            r--;
            while (r - l >= lowbit(r)) {
                ans = this.op.apply(ans, (T)arr[r]);
                r -= lowbit(r);
            }
        }
        return ans;
    }

    // O(log(n)), 符合单调性的更新
    public void update(int p, T v) {
        this.ori[p] = this.op.apply((T)this.ori[p], v);
        while (p <= n) {
            arr[p] = this.op.apply((T)arr[p], v);
            p += lowbit(p);
        }
    }

    // 操作时间复杂度, O(log(n)^2), 强制覆盖更新
    public void set(int p, T v) {
        this.ori[p] = v;
        for (; p <= n; p += lowbit(p)) {
            this.arr[p] = this.ori[p];
//            this.arr[p] = v;
            for (int i = 1; i < lowbit(p); i <<= 1) {
                this.arr[p] = this.op.apply((T)this.arr[p], (T)this.arr[p - i]);
            }
        }
    }

    int lowbit(int x) {
        return x & -x;
    }
}
```


