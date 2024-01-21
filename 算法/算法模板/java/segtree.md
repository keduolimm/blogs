

# RMQ 版本的线段树


```java []

import java.util.function.BiFunction;
import java.util.function.Function;

// RMQ 类型的 线段树
class RmqSegTree<T> {

    int l, r;
    T val;
    RmqSegTree<T> left, right;
    BiFunction<T, T, T> func;

    public RmqSegTree(int l, int r, Function<Integer, T> initializer, BiFunction<T, T, T> func) {
        this.l = l;
        this.r = r;
        this.func = func;
        if (l == r) {
            this.val = initializer.apply(l);
        } else {
            int m = l + (r - l) / 2;
            this.left = new RmqSegTree(l, m, initializer, func);
            this.right = new RmqSegTree(m + 1, r, initializer, func);
            this.val = func.apply(this.left.val, this.right.val);
        }
    }

    // 更新准寻 func原则
    public void update(int p, T val) {
        if (p < l || p > r) return;
        if (l == r) {
            this.val = func.apply(this.val, val);
        } else {
            int m = l + (r - l) / 2;
            if (p <= m) {
                this.left.update(p, val);
            } else {
                this.right.update(p, val);
            }
            this.val = func.apply(this.left.val, this.right.val);
        }
    }

    // 强制覆盖
    public void replace(int p, T val) {
        if (p < l || p > r) return;
        if (l == r) {
            this.val = val;
        } else {
            int m = l + (r - l) / 2;
            if (p <= m) {
                this.left.replace(p, val);
            } else {
                this.right.replace(p, val);
            }
            this.val = func.apply(this.left.val, this.right.val);
        }
    }

    public T query(int l, int r) {
        if (l <= this.l && r >= this.r) {
            return this.val;
        }

        int m = this.l + (this.r - this.l) / 2;
        if (l <= m && r > m) {
            T val1 = this.left.query(l, m);
            T val2 = this.right.query(m + 1, r);
            return func.apply(val1, val2);
        } else if (r <= m) {
            return this.left.query(l, r);
        } else if (l > m) {
            return this.right.query(Math.max(l, m + 1), r);
        }
        return this.val;
    }

}

```

---

# Lazy SegTree

```java []
public class LazySegTree {

    private int l, r;
    LazySegTree left, right;
    long v;
    long change;

    public LazySegTree(int l, int r, int[] arr) {
        this.l = l;
        this.r = r;
        if (l == r) {
            this.v = arr[l];
        } else {
            int m = l + (r - l) / 2;
            this.left = new LazySegTree(l, m, arr);
            this.right = new LazySegTree(m + 1, r, arr);
        }
    }

    public void update(int ul, int ur, int delta) {
        if (isLeaf()) {
            this.change += delta;
            this.v += this.change;
            this.change = 0;
            return;
        }
        if (this.l >= ul && this.r <= ur) {
            this.change += delta;
            return;
        }
        pushDown();

        int m = l + (r - l) / 2;
        if (ur <= m) {
            this.left.update(ul, ur, delta);
        } else if (ul > m) {
            this.right.update(ul, ur, delta);
        } else {
            this.left.update(ul, m, delta);
            this.right.update(m + 1, ur, delta);
        }
        pushUp();
    }

    public long query(int p) {
        if (isLeaf()) {
            if (this.change != 0) {
                this.v += this.change;
                this.change = 0;
            }
            return this.v;
        }
        pushDown();
        int m = l + (r - l) / 2;
        if (p <= m) {
            return this.left.query(p);
        } else {
            return this.right.query(p);
        }
    }

    void pushDown() {
        if (this.change != 0) {
            if (this.left != null) this.left.change += this.change;
            if (this.right != null) this.right.change += this.change;
            this.change = 0;
        }
    }

    void pushUp() {
    }

    boolean isLeaf() {
        return this.l == this.r;
    }

}
```