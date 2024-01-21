
# 牛客小白月赛86 解题报告 | 珂学家 | 最大子数组和变体 + lazy线段树&动态区间树

---
# 前言 

---

# 整体评价

终于回归小白月赛的内核了，希望以后也继续保持，^_^.

---

## [A. 水盐平衡](https://ac.nowcoder.com/acm/contest/73450/A)

思路: 模拟

题目保证没有浓度相等的情况

盐度 a/b， c/d 的比较关系

演变为 a*d, b*c 两者的大小关系

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    int t;
    cin >> t;
    while (t-- > 0) {
        int a, b, c, d;
        cin >> a >> b >> c >> d;
        if (a * d > b * c) {
            cout << "S" << endl;
        } else {
            cout << "Y" << endl;
        }
    }
    return 0;
}
```

---

## [B. 水平考试](https://ac.nowcoder.com/acm/contest/73450/B)

思路: 位运算

s, t可以转化为二进制形态

s为t的一个子集，其实表示为

然后 s & t == s 

或者 s | t == t

两者皆可

```python []
t = int(input())

def mask(s: str) -> int:
    r = 0
    for c in s:
        r |= 1 << (ord(c) - ord('A'))
    return r

for i in range(t):
    s, t = input(), input()
    a, b = mask(s), mask(t)
    if a & b == a:
        print (10)
    else:
        print (0)
```

---

## [C. 数组段数](https://ac.nowcoder.com/acm/contest/73450/C)

思路: 分段编码

给每个区间一个递增的编号

然后 id(e) - id(s) + 1

方法二, 利用区间前缀和作差，时间复杂度可以少一个log

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;
import java.util.TreeMap;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int n = sc.nextInt(), m = sc.nextInt();

        TreeMap<Integer, Integer> tree = new TreeMap<>();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        int segId = 0;
        int j = 0;
        while (j < n) {
            int k = j + 1;
            while (k < n && arr[k - 1] == arr[k]) {
                k++;
            }
            tree.put(j, segId++);
            j = k;
        }

        for (int i =0; i < m; i++) {
            int l = sc.nextInt() - 1, r = sc.nextInt() - 1;
            int res = tree.floorEntry(r).getValue() - tree.floorEntry(l).getValue() + 1;
            System.out.println(res);
        }

    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }

}
```

---

## [D. 剪纸游戏](https://ac.nowcoder.com/acm/contest/73450/D)

思路: 传统的3种方案

- DFS
- BFS
- 并查集

这边借助BFS实现

通过统计矩形的(minX, minY) -> (maxX, maxY)

然后统计 (maxY - minY + 1) * (maxX - minX + 1) == 连通图的个数

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.StringTokenizer;

public class Main {

    static final int[][] dirs = new int[][] {
            {-1, 0}, {1, 0}, {0, -1}, {0, 1}
    };

    static boolean check(int y, int x, char[][] grid, boolean[][] vis) {
        int h = grid.length, w = grid[0].length;

        int minY = y, minX = x;
        int maxY = y, maxX = x;
        int cnt = 1;

        Deque<int[]> deq = new ArrayDeque<>();
        deq.offer(new int[] {y, x});
        vis[y][x] = true;

        while (!deq.isEmpty()) {
            int[] pos = deq.poll();
            for (int i = 0; i < dirs.length; i++) {
                int ty = pos[0] + dirs[i][0];
                int tx = pos[1] + dirs[i][1];
                if (ty >= 0 && ty < h && tx >= 0 && tx < w) {
                    if (grid[ty][tx] == '.' && !vis[ty][tx]) {
                        vis[ty][tx] = true;
                        deq.offer(new int[] {ty, tx});
                        cnt++;

                        minY = Math.min(minY, ty);
                        minX = Math.min(minX, tx);
                        maxY = Math.max(maxY, ty);
                        maxX = Math.max(maxX, tx);
                    }
                }
            }
        }

        return minY == y && minX == x && (maxX - minX + 1) * (maxY - minY + 1) == cnt;
    }

    public static void main(String[] args) {
        AReader sc = new AReader();

        int h = sc.nextInt(), w = sc.nextInt();
        char[][] grids = new char[h][];
        for (int i = 0; i < h; i++) {
            grids[i] = sc.next().toCharArray();
        }

        int res = 0;
        boolean[][] vis = new boolean[h][w];
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (grids[i][j] == '.' && !vis[i][j]) {
                    if (check(i, j, grids, vis)) {
                        res++;
                    }
                }
            }
        }
        System.out.println(res);

    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

    }

}
```

----

## [E. 可口蛋糕](https://ac.nowcoder.com/acm/contest/73450/E)

思路: 双指针滑窗 + 最大子数组和变体

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();

        int n = sc.nextInt();
        long W = sc.nextLong();

        long[] wpre = new long[n + 1];
        long[] ws = new long[n];
        for (int i = 0; i < n; i++) {
            ws[i] = sc.nextInt();
            wpre[i + 1] = wpre[i] + ws[i];
        }

        long[] dpre = new long[n + 1];
        long[] ds = new long[n];
        for (int i = 0; i < n; i++) {
            ds[i] = sc.nextInt();
            dpre[i + 1] = dpre[i] + ds[i];
        }

        long res = Long.MIN_VALUE;
        long best = 0;
        int j = 0;

        for (int i = 0; i < n; i++) {
            while (j <= i && (wpre[i + 1] - wpre[j + 1]) >= W) {
                best = Math.max(best + ds[j], ds[j]);
                best = Math.max(0, best);
                j++;
            }

            if (j > 0) {
                res = Math.max(res, best + dpre[i + 1] - dpre[j]);
            }
        }
        System.out.println(res);
    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }
    }

}
```

----

## [F. 可口蛋糕](https://ac.nowcoder.com/acm/contest/73450/F)

思路: lazy线段树 + 珂朵莉树

lazy线段树用于维护 区间更新+点查询

珂朵莉树，则单纯的维护区间

当然这边用差分数组，可以简化lazy线段树(二阶树状数组)

对于区间更新，可以分为l, r+1的分裂和重新组织

这样的话，其分类讨论的情况就变少了。

```java []

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Map;
import java.util.StringTokenizer;
import java.util.TreeMap;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();

        int n = sc.nextInt(), q = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        LazySegTree segTree = new LazySegTree(0, n - 1, arr);

        // 1. 预处理一下
        long acc = 0; // 当前最大的值
        TreeMap<Integer, Integer> tree = new TreeMap<>();
        int j = 0;
        while (j < n) {
            int k = j + 1;
            while (k < n && arr[k - 1] + 1 == arr[k]) {
                k++;
            }
            acc += (long)(k - j) * (k - j);
            tree.put(j, k - 1);
            j = k;
        }

        // 和区间相关的题
        for (int i = 0; i < q; i++) {
            int l = sc.nextInt() - 1, r = sc.nextInt() - 1;
            int d = sc.nextInt();

            if (d == 0) {
                System.out.println(acc);
            } else {
                Map.Entry<Integer, Integer> ent = tree.floorEntry(l);
                segTree.update(l, r, d);
                
                if (ent.getKey() == l) {
                    if (l > 0 && segTree.query(l - 1) + 1 == segTree.query(l)) {
                        Map.Entry<Integer, Integer> ev1 = tree.floorEntry(l - 1);

                        int d1 = ev1.getValue() - ev1.getKey() + 1;
                        int d2 = ent.getValue() - ent.getKey() + 1;
                        acc = acc - (long)d1 * d1 - (long)d2 * d2;
                        acc = acc + (long)(d1 + d2) * (d1 + d2);

                        tree.remove(ev1.getKey());
                        tree.remove(ent.getKey());
                        tree.put(ev1.getKey(), ent.getValue());
                    }
                } else {
                    int d1 = l - ent.getKey();
                    int d2 = ent.getValue() - l + 1;

                    int span = ent.getValue() - ent.getKey() + 1;

                    acc = acc - (long)span * span;
                    acc = acc + (long)d1 * d1 + (long)d2 * d2;

                    tree.remove(ent.getKey());
                    if (ent.getKey() < l) {
                        tree.put(ent.getKey(), l - 1);
                    }
                    tree.put(l, ent.getValue());
                }

                // *)
                Map.Entry<Integer, Integer> ent2 = tree.floorEntry(r);
                if (ent2.getValue() == r) {
                    if (r + 1 < n && segTree.query(r) + 1 == segTree.query(r + 1)) {
                        Map.Entry<Integer, Integer> ev1 = tree.floorEntry(r + 1);

                        int d1 = ent2.getValue() - ent2.getKey() + 1;
                        int d2 = ev1.getValue() - ev1.getKey() + 1;

                        acc = acc - (long)d1 * d1 - (long)d2 * d2;
                        acc = acc + (long)(d1 + d2) * (d1 + d2);

                        tree.remove(ev1.getKey());
                        tree.remove(ent2.getKey());
                        tree.put(ent2.getKey(), ev1.getValue());
                    }
                } else {
                    int d2 = ent2.getValue() - r;
                    int d1 = r - ent2.getKey() + 1;

                    int span = ent2.getValue() - ent2.getKey() + 1;

                    acc = acc - (long)span * span;
                    acc = acc + (long)d1 * d1 + (long)d2 * d2;

                    tree.put(ent2.getKey(), r);
                    if (ent2.getValue() > r) {
                        tree.put(r + 1, ent2.getValue());
                    }
                }
                System.out.println(acc);
            }
        }

    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }


    static
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

}
```

----

# 写在最后


