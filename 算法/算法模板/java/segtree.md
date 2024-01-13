

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