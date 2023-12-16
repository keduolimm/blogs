# 第 362 场周赛 解题报告 | 珂学家 | 模板场 + 最大流最小费用 + 矩阵幂优化的DP


---

# 前言

![image.png](https://pic.leetcode.cn/1694430409-LPphsZ-image.png)


---

# 整体评价

力扣周赛越来越难了，也越来越板了。现在AK都成奢求了，T_T.

![image.png](https://pic.leetcode.cn/1694439875-qNeWbg-image.png){:align=left}


---

## [T1. 与车相交的点](https://leetcode.cn/contest/weekly-contest-362/problems/points-that-intersect-with-cars/)

差分入门题

尝试写个简易版珂朵莉树玩玩，这边只合并不分裂。

如果题目改造为设计题
- 实时添加区间
- 实时查询区间总包含的数

那珂朵莉树就有优势


```java [group1-珂朵莉树]
class Solution {
    public int numberOfPoints(List<List<Integer>> nums) {
        TreeMap<Integer, int[]> chtholly = new TreeMap<>();
        for (var num: nums) {
            int s = num.get(0), e = num.get(1);
            Map.Entry<Integer, int[]> ent = null;
            while ((ent = chtholly.floorEntry(e)) != null) {
                int[] cur = ent.getValue();
                if (cur[1] < s) break;
                chtholly.remove(ent.getKey());
                s = Math.min(s, cur[0]);
                e = Math.max(e, cur[1]);
            }
            // 区间左端点作为key
            chtholly.put(s, new int[] {s, e});
        }
        // 区间累加和
        return chtholly.values().stream().mapToInt(x -> x[1] - x[0] + 1).sum();
    }
}
```
```java [group1-差分]
class Solution {
    public int numberOfPoints(List<List<Integer>> nums) {
        int[] diff = new int[102];
        for (var e: nums) {
            diff[e.get(0)]++;
            diff[e.get(1) + 1]--;
        }
        
        int ans = 0;
        int acc = 0;
        for (int i = 1; i <= 100; i++) {
            acc += diff[i];
            if (acc > 0) {
                ans++;
            }
        }
        return ans;
    }
}
```

---

## [T2. 判断能否在给定时间到达单元格](https://leetcode.cn/contest/weekly-contest-362/problems/determine-if-a-cell-is-reachable-at-a-given-time/)

这题本质上可以转化为如下约束方程

$$
\left\{\begin{matrix}
fx=sx + dx_1 + ... + dx_t \\  
fy=sy + dy_1 + ... + dy_t \\
dx_i + dy_i \ne 0
\end{matrix}\right.
$$

显而易见，${abs(fx - sx) \gt t \parallel  abs(fy - sy) \gt t}$ 一定无解

但这题，还有个特殊之处，就在于 sx=fx, sy=fy, t=1, 不存在

其他都能构造

```java
class Solution {
    public boolean isReachableAtTime(int sx, int sy, int fx, int fy, int t) {
        long d1 = Math.abs(sx - fx);
        long d2 = Math.abs(sy - fy);
        
        if (d1 == 0 && d2 == 0) {
            return t != 1;
        }
        
        return d1 <= t && d2 <= t;
    }
}
```


---
## [T3. 将石头分散到网格图的最少移动次数](https://leetcode.cn/contest/weekly-contest-362/problems/minimum-moves-to-spread-stones-over-grid/)

如果把每个格子，0的格子和大于1的格子(拆解为最小粒度)提取出来，这是一道很经典的二分图带权最大匹配模型。

如果不限定矩阵的大小，那对应的算法也就不同了。

![image.png](https://pic.leetcode.cn/1694441202-tCjXNv-image.png){:align=left}{:width=600px}


- 3x3矩阵

那最多8个原点/目标点，可以固定原定，然后排列组合，从而计算最小的代价

采用普通DFS搜索

还可以继续优化，使用计数DFS

```java [group3-普通DFS]
class Solution {
    
    List<int[]> need = new ArrayList<>();
    List<int[]> more = new ArrayList<>();
    
    int gAns = Integer.MAX_VALUE;
    
    void dfs(int i, int cost, int state) {
        if (i >= need.size()) {
            gAns = Math.min(gAns, cost);
            return;
        }
        // 剪枝
        if (cost >= gAns) return;
        for (int j = 0; j < more.size(); j++) {
            if (((1 << j) & state) != 0) continue;
            int delta = Math.abs(need.get(i)[0] - more.get(j)[0]) 
                        + Math.abs(need.get(i)[1] - more.get(j)[1]);
            dfs(i + 1, cost + delta, state | (1 << j));
        }
    }
    
    public int minimumMoves(int[][] grid) {
        // 转化模型
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[i].length; j++) {
                if (grid[i][j] == 0) need.add(new int[] {i, j});
                else if (grid[i][j] > 1) {
                    // 拆分为粒度1的目标格子
                    for (int k = 0; k < grid[i][j] - 1; k++) {
                        more.add(new int[] {i, j});
                    }
                }
            }
        }
        dfs(0, 0, 0);
        return gAns;
    }
    
}
```
```java [group3-计数DFS]
class Solution {
    
    List<int[]> need = new ArrayList<>();
    List<int[]> more = new ArrayList<>();
    
    int gAns = Integer.MAX_VALUE;
    
    void dfs(int i, int cost) {
        if (i >= need.size()) {
            gAns = Math.min(gAns, cost);
            return;
        }
        // 剪枝
        if (cost >= gAns) return;
        for (int j = 0; j < more.size(); j++) {
            int[] t = more.get(j);
            if (t[2] > 0) {
                t[2]--;
                int delta = Math.abs(need.get(i)[0] - more.get(j)[0])
                        + Math.abs(need.get(i)[1] - more.get(j)[1]);
                dfs(i + 1, cost + delta);
                t[2]++;
            }
        }
    }
    
    public int minimumMoves(int[][] grid) {
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[i].length; j++) {
                if (grid[i][j] == 0) need.add(new int[] {i, j});
                else if (grid[i][j] > 1) {
                    more.add(new int[] {i, j, grid[i][j] - 1});
                }
            }
        }
        dfs(0, 0);
        return gAns;
    }
    
}
```

- 4x4矩阵

如果是4x4，感觉dfs有点吃力

这个时候，可以使用状压DP来求解

也是固定一列，然后排列另一列

设$opt[s]$, 表示s中二进制$2^i$对标格子已被挑选

${opt[t]= \max_{\ k\in t, k\notin s} opt[s] + Manhattan(bitCount(s), k) }$

```java
class Solution {
    
    List<int[]> need = new ArrayList<>();
    List<int[]> more = new ArrayList<>();
    
    public int minimumMoves(int[][] grid) {

        // 配对           
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (grid[i][j] == 0) {
                    need.add(new int[] {i, j});
                } else if (grid[i][j] > 1) {
                    for (int k = 0; k < grid[i][j] - 1; k++) {
                        more.add(new int[] {i, j});
                    }
                }
            }
        }
        
        int m = need.size();
        int range = 1 << m;
        int[] opt = new int[range];
        Arrays.fill(opt, Integer.MAX_VALUE / 10);
                               
        opt[0] = 0;
        
        for (int i = 0; i < range; i++) {
            int cnt = Integer.bitCount(i);
            for (int j = 0; j < m; j++) {
                if (((1 << j) & i) != 0) continue;
                
                int d = Math.abs(need.get(cnt)[0] - more.get(j)[0]) + Math.abs(need.get(cnt)[1] - more.get(j)[1]);
                
                if (opt[i | (1 << j)] > opt[i] + d) {
                    opt[i | (1 << j)] = opt[i] + d;
                }
                
            }
        }
        
        return opt[range - 1];
        
    }
    
}
```

- 10x10矩阵

这个时候，只能借助网络流来求解了

最大流最小费用的模板，抄自 uwi
```java [group2-代码]
class Solution {

    public int minimumMoves(int[][] grid) {

        List<int[]> need = new ArrayList<>();
        List<int[]> more = new ArrayList<>();

        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[i].length; j++) {
                if (grid[i][j] == 0) need.add(new int[] {i, j});
                else if (grid[i][j] > 1) {
                    more.add(new int[] {i, j, grid[i][j] - 1});
                }
            }
        }

        List<MinCostFlowAlgorithm.Edge> edges = new ArrayList<>();

        // 引入虚拟原点和目标点
        int source = need.size() + more.size(), target = source + 1;
        for (int i = 0; i < need.size(); i++) {
            edges.add(new MinCostFlowAlgorithm.Edge(source, i, 1, 0));
        }
        // 构建中间层的有向边
        for (int i = 0; i < need.size(); i++) {
            for (int j = 0; j < more.size(); j++) {
                int cost = Math.abs(need.get(i)[0] - more.get(j)[0]) + Math.abs(need.get(i)[1] - more.get(j)[1]);
                edges.add(new MinCostFlowAlgorithm.Edge(i, j + need.size(), 1, cost));
            }
        }
        for (int j = 0; j < more.size(); j++) {
            edges.add(new MinCostFlowAlgorithm.Edge(j + need.size(), target, more.get(j)[2], 0));
        }

        MinCostFlowAlgorithm mcf = new MinCostFlowAlgorithm();
        return (int)mcf.solveMinCostFlow(
                MinCostFlowAlgorithm.compileWD(target + 1, edges),  // 转换为数组
                source,       // 原节点
                target,       // 目标节点
                need.size()   // 流量
        );
    }

}
```

```java [group2-最大流最小费用模板]
static class MinCostFlowAlgorithm {

    static
    public class Edge {
        public int from, to;
        public int capacity;
        public long cost;
        public int flow;
        public Edge complement;
        // public int iniflow;

        public Edge(int from, int to, int capacity, long cost) {
            this.from = from;
            this.to = to;
            this.capacity = capacity;
            this.cost = cost;
        }

        @Override
        public String toString() {
            return "Edge [from=" + from + ", to=" + to + ", capacity="
                    + capacity + ", cost=" + cost + "]";
        }
    }

    public static Edge[][] compileWD(int n, List<Edge> edges) {
        Edge[][] g = new Edge[n][];
        // cloning
        for (int i = edges.size() - 1; i >= 0; i--) {
            Edge origin = edges.get(i);
            Edge clone = new Edge(origin.to, origin.from, origin.capacity, -origin.cost);
            clone.flow = origin.capacity;
            clone.complement = origin;
            origin.complement = clone;
            edges.add(clone);
        }

        int[] p = new int[n];
        for (Edge e : edges) p[e.from]++;
        for (int i = 0; i < n; i++) g[i] = new Edge[p[i]];
        for (Edge e : edges) g[e.from][--p[e.from]] = e;
        return g;
    }

    public long solveMinCostFlow(Edge[][] g, int source, int sink, long all) {
        int n = g.length;
        long mincost = 0;
        long[] potential = new long[n];

        final long[] d = new long[n];
        MinHeapL q = new MinHeapL(n);
        while (all > 0) {
            // shortest path src->sink
            Edge[] inedge = new Edge[n];
            Arrays.fill(d, Long.MAX_VALUE / 2);
            d[source] = 0;
            q.add(source, 0);
            while (q.size() > 0) {
                int cur = q.argmin();
                q.remove(cur);
                for (Edge ne : g[cur]) {
                    if (ne.capacity - ne.flow > 0) {
                        long nd = d[cur] + ne.cost + potential[cur] - potential[ne.to];
                        if (d[ne.to] > nd) {
                            inedge[ne.to] = ne;
                            d[ne.to] = nd;
                            q.update(ne.to, nd);
                        }
                    }
                }
            }

            if (inedge[sink] == null) break;

            long minflow = all;
            long sumcost = 0;
            for (Edge e = inedge[sink]; e != null; e = inedge[e.from]) {
                if (e.capacity - e.flow < minflow) minflow = e.capacity - e.flow;
                sumcost += e.cost;
            }
            mincost += minflow * sumcost;
            for (Edge e = inedge[sink]; e != null; e = inedge[e.from]) {
                e.flow += minflow;
                e.complement.flow -= minflow;
            }

            all -= minflow;
            for (int i = 0; i < n; i++) {
                potential[i] += d[i];
            }
        }
        return mincost;
    }

    public class MinHeapL {
        public long[] a;
        public int[] map;
        public int[] imap;
        public int n;
        public int pos;
        public static long INF = Long.MAX_VALUE;

        public MinHeapL(int m) {
            n = Integer.highestOneBit((m + 1) << 1);
            a = new long[n];
            map = new int[n];
            imap = new int[n];
            Arrays.fill(a, INF);
            Arrays.fill(map, -1);
            Arrays.fill(imap, -1);
            pos = 1;
        }

        public long add(int ind, long x) {
            int ret = imap[ind];
            if (imap[ind] < 0) {
                a[pos] = x;
                map[pos] = ind;
                imap[ind] = pos;
                pos++;
                up(pos - 1);
            }
            return ret != -1 ? a[ret] : x;
        }

        public long update(int ind, long x) {
            int ret = imap[ind];
            if (imap[ind] < 0) {
                a[pos] = x;
                map[pos] = ind;
                imap[ind] = pos;
                pos++;
                up(pos - 1);
            } else {
                a[ret] = x;
                up(ret);
                down(ret);
            }
            return x;
        }

        public long remove(int ind) {
            if (pos == 1) return INF;
            if (imap[ind] == -1) return INF;

            pos--;
            int rem = imap[ind];
            long ret = a[rem];
            map[rem] = map[pos];
            imap[map[pos]] = rem;
            imap[ind] = -1;
            a[rem] = a[pos];
            a[pos] = INF;
            map[pos] = -1;

            up(rem);
            down(rem);
            return ret;
        }

        public long min() {
            return a[1];
        }

        public int argmin() {
            return map[1];
        }

        public int size() {
            return pos - 1;
        }

        public long get(int ind) {
            return a[imap[ind]];
        }

        private void up(int cur) {
            for (int c = cur, p = c >>> 1; p >= 1 && a[p] > a[c]; c >>>= 1, p >>>= 1) {
                long d = a[p];
                a[p] = a[c];
                a[c] = d;
                int e = imap[map[p]];
                imap[map[p]] = imap[map[c]];
                imap[map[c]] = e;
                e = map[p];
                map[p] = map[c];
                map[c] = e;
            }
        }

        private void down(int cur) {
            for (int c = cur; 2 * c < pos; ) {
                int b = a[2 * c] < a[2 * c + 1] ? 2 * c : 2 * c + 1;
                if (a[b] < a[c]) {
                    long d = a[c];
                    a[c] = a[b];
                    a[b] = d;
                    int e = imap[map[c]];
                    imap[map[c]] = imap[map[b]];
                    imap[map[b]] = e;
                    e = map[c];
                    map[c] = map[b];
                    map[b] = e;
                    c = b;
                } else {
                    break;
                }
            }
        }
    }
}
```


--- 

## [T4. 字符串转换](https://leetcode.cn/contest/weekly-contest-362/problems/string-transformation/)

在字符串S寻找一个切分点i，使得

$$S[i+1:] + S[0:i+1] == T$$

这有很多解决算法。

这边采用字符串的Rolling Hash来求解。

假定满足切分点的个数为 $p$, 不满足的点个数为$n-p$

方案总数又如何求解呢？

由于切割范围$0\lt l \lt n$, 这边可以做一个优化点，就是状态压缩

$f=不匹配态，g=匹配态$

可以找到转移方程

$$
\left\{\begin{matrix}
f_i & = & (n-p-1) * f_{i-1} + (n - p) * g_{i-1} \\  
g_i & = & p * f_{i-1} + (p - 1) * g_{i - 1}
\end{matrix}\right.
$$

因为转移k太大了，所以可以转化为矩阵幂来优化

$$
\begin{bmatrix}
f_k\\
g_k
\end{bmatrix}
=
\begin{bmatrix}
n - p - 1, n - p\\
p, p - 1
\end{bmatrix}^k
\times
\begin{bmatrix}
f_0\\
g_0
\end{bmatrix}
$$

不过这里的初始状态比较特殊

如果 $S==T$, 那么$f_0=0, g_0=1$
否则 $S==T$, 那么$f_0=1, g_0=0$

```java
class Solution {

    static class StringHash {
        char[] str;
        long p, mod;

        int n;
        long[] pre; // hash前缀和
        long[] pow; // p的幂次

        public StringHash(String s, int p, long mod) {
            this.str = s.toCharArray();
            this.p = p;
            this.mod = mod;

            this.n = str.length;
            pre = new long[n + 1];
            pow = new long[n + 1];

            pow[0] = 1;
            for (int i = 1; i <= n; i++) {
                pow[i] = pow[i - 1] * p % mod;
            }
            for (int i = 0; i < str.length; i++) {
                pre[i + 1] = (pre[i] * p % mod + str[i]) % mod;
            }
        }

        long query(int l, int r) {
            long res = pre[r + 1] - pre[l] * pow[r - l + 1] % mod;
            return (res % mod + mod) % mod;
        }

        long rotate(int l) {
            if (l < 0 || l >= str.length - 1) {
                return query(0, str.length - 1);
            } else {
                long h1 = query(0, l);
                long h2 = query(l + 1, str.length - 1);
                return (h2 * pow[l + 1] % mod + h1) % mod;
            }
        }

        static long evaluate(String s, int p, long mod) {
            long h = 0;
            for (char c: s.toCharArray()) {
                h = (h * p % mod + c) % mod;
            }
            return h;
        }
    }

    static class Matrix {
        long[][] arr;
        int r, c;
        Matrix(long[][] arr) {
            this.arr = arr;
            this.r = arr.length;
            this.c = arr[0].length;
        }

        Matrix mul(Matrix other, long mod) {
            int nr = this.r, nc = other.c;
            long[][] res = new long[nr][nc];
            for (int i = 0; i < nr; i++) {
                for (int j = 0; j < nc; j++) {
                    long temp = 0;
                    for (int k = 0; k < c; k++) {
                        temp = (temp + arr[i][k] * other.arr[k][j] % mod) % mod;
                    }
                    res[i][j] = temp;
                }
            }
            return new Matrix(res);
        }

        static Matrix quickPow(Matrix base, long k, long mod) {
            Matrix r = new Matrix(new long[][] {
                    {1, 0},
                    {0, 1}
            });
            while (k > 0) {
                if (k % 2 == 1) {
                    r = r.mul(base, mod);
                }
                k /= 2;
                base = base.mul(base, mod);
            }
            return r;
        }
    }

    public int numberOfWays(String s, String t, long k) {
        int z = 13;
        long mod = 10_0000_0007l;
        int n = s.length();

        // 单hash
        StringHash h = new StringHash(s, z, mod);
        long code = StringHash.evaluate(t, z, mod);
        int first = (code == h.query(0, n - 1)) ? 1 : 0;

        int p = first;
        for (int i = 0; i < n - 1; i++) {
            if (code == h.rotate(i)) {
                p++;
            }
        }

        // 转移矩阵
        Matrix base = new Matrix(new long[][] {
            {n - p - 1, n - p},
                {p, p - 1}
        });
        // 快速幂
        Matrix r = Matrix.quickPow(base, k, mod);
        // 矩阵求解
        Matrix ans = r.mul(new Matrix(new long[][] {{1 - first}, {first}}), mod);

        return (int)ans.arr[1][0];
    }

}
```

---

# 写在最后

![image.png](https://pic.leetcode.cn/1694430336-UAJoAq-image.png)
