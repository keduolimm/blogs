# 第 366 场周赛 解题报告 | 珂学家 | 状态机DP + 贪心


---
# 前言


![image.png](https://pic.leetcode.cn/1696740015-prLbKH-image.png)

当你重新踏上旅途之后，一定要记得旅途本身的意义。

----

# 整体评价

T4好像哪里做过，按位拆分贪心即可. T3真的是大魔王，可惜又看错题了, T_T。

T3 完全是通过错误的case，从而推测算法，不知道对不对.

T3 补充 最大流最小费用解法。

----

## [T1. 分类求和并作差](https://leetcode.cn/contest/weekly-contest-366/problems/divisible-and-non-divisible-sums-difference/)


可以先算1~n的累计和

$$ s=(n+1)*n/2$$

然后算m的倍数累加和

$$ t=n/m, 表示在[1, n],有t个m的倍数$$

$$ s1 = (t+1) * t / 2 * m, 为m倍数的累加和$$
$$ res = (s - s1) - s1 = s - 2 * s1$$

```java
public class Solution {
    public static int differenceOfSums(int n, int m) {
        // 1~n的累加和
        int s = (n + 1) * n / 2; 
        // m的倍数的个数
        int t = n / m;
        /// s1表示m的倍数的累计和
        int s1 = (t + 1) * t / 2 * m;
        
        return s - 2 * s1;
    }
}
```

---

## [T2. 最小处理时间](https://leetcode.cn/contest/weekly-contest-366/problems/minimum-processing-time/)

模拟+贪心

让先空闲的cpu跑时间长的task， 这样完成所有的时间最短。

```java
public class Solution {
    public static int minProcessingTime(List<Integer> processorTime, List<Integer> tasks) {
        
        Collections.sort(processorTime, Comparator.comparingInt(x -> x));
        Collections.sort(tasks, Comparator.comparingInt(x -> -x));
        
        int ans = 0;
        for (int i = 0; i < processorTime.size(); i++) {
            for (int j = 0; j < 4; j++) {
                ans = Math.max(ans, processorTime.get(i) + tasks.get(i * 4 + j));
            }
        }
        
        return ans;
        
    }
}
```

----

## [T3. 执行操作使两个字符串相等](https://leetcode.cn/contest/weekly-contest-366/problems/apply-operations-to-make-two-strings-equal/)

重要的事说三遍
> 反转 不是 swap，而是reverse
> 反转 不是 swap，而是reverse
> 反转 不是 swap，而是reverse

这题本质

$$一般图最大权匹配$$

这边给出三种不同思路的解法

- 区间DP
- 状态机DP
- 二分图最大权匹配

---

### 区间DP

提取需要反转的位子索引，并构建一个序列

可以观察发现

$$ \color{Pink} 两两匹配不存在交叉$$

![image.png](https://pic.leetcode.cn/1696948756-gWOYPu-image.png)


所以它的匹配结构一定是

- 嵌套结构
  $f(s, e)=f(s + 1, e - 1) + min(pos[e] - pos[s], x)$
- 左右并列结构
  ${f(s, e)=\min_{i \in [s,e )} (f(s, i) + f(i+1,e)) }$

这种形式，就是最经典的区间DP形态

```java
class Solution {
    
    static int inf = Integer.MAX_VALUE / 10;
    
    Integer[][] memo;
    
    // 记忆化搜索
    int dfs(int s, int e, BiFunction<Integer, Integer, Integer> evalutor) {         
        if (memo[s][e] != null) {
            return memo[s][e];
        }
        
        if (s == e) return inf;
        if (s + 1 == e) {
            return memo[s][e] = evalutor.apply(s, e);
        }
        
        // S=(S) 嵌套结构
        int res = dfs(s + 1, e - 1, evalutor) + evalutor.apply(s, e);
        
        // S=SS 左右结构, i是切割点
        for (int i = s; i < e; i++) {
            int tmp = dfs(s, i, evalutor) + dfs(i + 1, e, evalutor);
            res = Math.min(res, tmp);
        }
        
        return memo[s][e] = res;
        
    }
 
    public int minOperations(String s1, String s2, int x) {

        // 提取不相等的位子序列
        List<Integer> seq = IntStream.range(0, s1.length())
            .filter(i -> s1.charAt(i) != s2.charAt(i))
            .boxed()
            .collect(Collectors.toList());
        
        if (seq.size() == 0) return 0;
        
        int m = seq.size();
        this.memo = new Integer[m][m];
        
        // 定义评估函数
        BiFunction<Integer, Integer, Integer> evalutor = 
            (a, b) -> Math.min(Math.abs(seq.get(a) - seq.get(b)), x);
        
        // 记忆化搜索
        int res = dfs(0, m - 1, evalutor);
        return res >= inf ? -1 : res;

    }

}
```

时间复杂度为$O(n^3)$

---

### 状态机DP

对于两个$i, j$，如果他们配对，那么代价为

$$ cost = min(abs(pos[j] - pos[i]), x) $$

抽取不同的位置构建序列$arr$，长度为$m$

引入状态机DP

$$令 opt[i][j] 为前i项中, j项尚未匹配的最优解 $$
$$ opt[i][m], m为特殊状态, 表示全i项都配对后的最优解 $$

状态转移

$$  opt[i][m] = \min_{j\in[0, i)}opt[i - 1][j] + min(abs(pos[i] - pos[j]), x) \tag{1} $$
$$  opt[i][j] = opt[i - 2][j] + min(abs(pos[i] - pos[i - 1]), x),  j\in[0, i) \tag{2} $$
$$  opt[i][i] = opt[i - 2][n] \tag{3} $$

关于不可解

其实可以通过奇偶性，来快速筛选

```java
class Solution {
 
    public int minOperations(String s1, String s2, int x) {

        char[] str1 = s1.toCharArray();
        char[] str2 = s2.toCharArray();

        int n = str1.length;
        // 提取需要变更的位子，构建一个新序列
        List<Integer> diff = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (str1[i] != str2[i]) {
                diff.add(i);
            }
        }
        if (diff.size() == 0) return 0;

        int inf = Integer.MAX_VALUE / 10;
        int m = diff.size();
        int[][] opt = new int[m][m + 1];
        for (int i = 0; i < m; i++) {
            Arrays.fill(opt[i], inf);
        }
        opt[0][0] = 0;

        for (int i = 1; i < m; i++) {
            // 全部匹配
            for (int j = 0; j < i; j++) {
                opt[i][m] = Math.min(opt[i][m], opt[i - 1][j] + Math.min(diff.get(i) - diff.get(j), x));
            }

            // 有一项没匹配
            if (i >= 2) {
                for (int j = 0; j < i; j++) {
                    opt[i][j] = Math.min(opt[i][j], opt[i - 2][j] + Math.min(diff.get(i) - diff.get(i - 1), x));
                }
                opt[i][i] = Math.min(opt[i][i], opt[i - 1][m]);
            }
        }

        return opt[m - 1][m] == inf ? -1 : opt[m - 1][m];

    }

}
```

时间复杂度为$O(n^2)$

---

### 二分图的最大权匹配

这题较特殊，可以由

$$一般图的最大权匹配问题 \Rightarrow 二分图的最大权匹配模型$$

按奇偶组织成2个集合，匹配边的端点一个在奇数集合S1, 另一个在偶数集合S2.

![image.png](https://pic.leetcode.cn/1696779502-iXZlnG-image.png)

感谢 [@ItzDesert](/u/desert-fu/) , 给出的推导和证明。

本质还是匹配结构，不存在交叉，要么嵌套，要么左右并列，导致匹配必然一端是奇数索引，一端是偶数索引。


```java [group-代码]
class Solution {
 
    public int minOperations(String s1, String s2, int x) {

        char[] str1 = s1.toCharArray();
        char[] str2 = s2.toCharArray();

        int n = str1.length;
        // 提取需要变更的位子，构建一个新序列
        List<Integer> diff = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (str1[i] != str2[i]) {
                diff.add(i);
            }
        }
        if (diff.size() == 0) return 0;
        
        int m = diff.size();
        if (m % 2 == 1) return -1;

        
        int source = m;
        int target = m + 1;
        int flow = m / 2;
    
        List<MinCostFlowAlgorithm.Edge> edges = new ArrayList<>();
        
        // 把奇偶 组织成 二分图
        // 那这个图转化为 二分图最大权匹配问题
        // 用最大流最小费用求解
        for (int i = 0; i < diff.size(); i++) {
            if (i % 2 == 0) {
                edges.add(new MinCostFlowAlgorithm.Edge(source, i, 1, 0));
            } else {
                edges.add(new MinCostFlowAlgorithm.Edge(i, target, 1, 0));
            }
        }
        
        for (int i = 0; i < diff.size(); i += 2) {
            for (int j = 1; j < diff.size(); j += 2) {
                edges.add(new MinCostFlowAlgorithm.Edge(i, j, 1, Math.min(Math.abs(diff.get(j) - diff.get(i)), x)));
            }
        }
        
        MinCostFlowAlgorithm mcfa = new MinCostFlowAlgorithm();
        return (int)mcfa.solveMinCostFlow(
            MinCostFlowAlgorithm.compileWD(target + 1, edges), 
            source,  // 虚拟源点 
            target,  // 虚拟终点
            flow     // 指定流大小
        );
        
    }
    
}
```

```java [group-最大流最小费用模板]
// 最大流最小费模型
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

AC代码的提交链接: [请点这里](https://leetcode.cn/problems/apply-operations-to-make-two-strings-equal/submissions/472410818/)

---

## [T4. 对数组执行操作使平方和最大](https://leetcode.cn/contest/weekly-contest-366/problems/apply-operations-on-array-to-maximize-sum-of-squares/)

按位拆分，然后重组，贪心

因为如下公式恒成立

$$ (2^i + 2 ^ j)^2 + 0 > (2^i)^2 + (2^j)^2, i\ne j $$

因此尽量把1聚合，这样的组合，其平方和一定最大

```java
class Solution {
    
    public int maxSum(List<Integer> nums, int k) {

        int[] hash = new int[30];
        for (int v: nums) {
            for (int i = 0; i < 30; i++) {
                if (((1 << i) & v) != 0) {
                    hash[i]++;
                }
            }
        }
        
        long mod = 10_0000_0007l;
        long ans = 0;
        int n = Math.min(k, nums.size());
        for (int i = 0; i < n; i++) {
            int tmp = 0;
            for (int j = 29; j >= 0; j--) {
                if (hash[j] > 0) {
                    hash[j]--;
                    tmp += (1 << j);
                }
            }
            ans = (ans + (long)tmp * tmp % mod) % mod;
        }
        
        return (int)ans;
        
    }
    
}
```

---

# 写在最后

致敬原神

![image.png](https://pic.leetcode.cn/1696739961-FfsTsH-image.png)
