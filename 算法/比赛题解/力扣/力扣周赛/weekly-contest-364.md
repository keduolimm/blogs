# 第 364 场周赛 解题报告 | 珂学家 | 前后缀分解&单调栈 + 换根DP



---
# 前言

![image.png](https://pic.leetcode.cn/1695529374-KDYJnA-image.png)

这个世界，只需要一把剑就可以去往任何地方。

---
# 整体评价

T3 虽然套路，但是有一定的思维难度，T4难点不在换根，还在于计算，如果不优化遇到菊花图会TLE。

补充了并查集做法，感觉这个更容易理解。

---
## [T1. 最大二进制奇数](https://leetcode.cn/problems/maximum-odd-binary-number/)

简单统计0,1的个数 cnt0,cnt1

然后贪心分三段

- (cnt1 - 1)个1
- cnt0个0
- 最后一个拖尾的1

```java
public class Solution {
    public String maximumOddBinaryNumber(String s) {
        int cnt1 = (int)s.chars().filter(x -> x == (int)('1')).count();
        int cnt0 = s.length() - cnt1;
        return "1".repeat(cnt1 - 1) + "0".repeat(cnt0) + "1";
    }
}
```

---
## [T2. 美丽塔 I](https://leetcode.cn/contest/weekly-contest-364/problems/beautiful-towers-i/)

和T3一起讲

---

## [T3. 美丽塔 II](https://leetcode.cn/contest/weekly-contest-364/problems/beautiful-towers-ii/)

因为题目和山峰有关，而确定山峰点，左右侧限制就是单调的。

所以很自然的想法就是

> 前后缀拆解 + 枚举山峰点

但是这个前后缀拆解有点困难，他并不是常见的前缀和/DP形态的。

它有 范围限制，以及前后两个大小的限制。

回到主题，如果一侧是单调的，那为何我们不试试单调栈呢？

```
定义栈内元素，tuple = (高度，区间left, 区间right)
```

同时外面在维护一个全局的最优和，这样就可以计算出 前缀最优

同理，我们逆向求的 后缀最优

不过最后的结果，需要注意下

$$ ans = \max_{i\in[0, n)} \{ pre[i] + suf[i] - maxHeight[i] \} $$

时间复杂度为$O(n)$, 因为单调栈均摊操作。

```java
class Solution {

    public long maximumSumOfHeights(List<Integer> maxHeights) {
        // 前后缀拆解 + 枚举山峰点
        int n = maxHeights.size();
        long[] pre = new long[n];
        long[] suf = new long[n];

        // 计算前缀(左侧)最多
        Deque<long[]> mono1 = new ArrayDeque<>();
        long acc1 = 0; // 维护累计和
        for (int i = 0; i < n; i++) {
            int v = maxHeights.get(i);
            long s = i;
            while (!mono1.isEmpty() && mono1.peekLast()[0] > v) {
                long[] cur = mono1.pollLast();
                acc1 -= cur[0] * (cur[2] - cur[1] + 1); // 动态减去
                s = cur[1];
            }
            acc1 += (i - s + 1) * v; // 动态加上
            mono1.offerLast(new long[]{v, s, i});

            pre[i] = acc1;
        }

        // 计算后缀(右侧)最多
        Deque<long[]> mono2 = new ArrayDeque<>();
        long acc2 = 0; 
        for (int i = n - 1; i >= 0; i--) {
            int v = maxHeights.get(i);
            long s = i;
            while (!mono2.isEmpty() && mono2.peekLast()[0] > v) {
                long[] cur = mono2.pollLast();
                acc2 -= cur[0] * (cur[2] - cur[1] + 1);
                s = cur[2];
            }
            acc2 += (s - i + 1) * v;
            mono2.offerLast(new long[]{v, i, s});

            suf[i] = acc2;
        }

        // 最后的结果
        long ans = 0;
        for (int i = 0; i < n; i++) {
            ans = Math.max(ans, suf[i] + pre[i] - maxHeights.get(i));
        }

        return ans;
    }

}
```


---

## [T4. 统计树中的合法路径数目](https://leetcode.cn/contest/weekly-contest-364/)

因为美丽路径(仅包含一个质数点)，所以我们可以从质数点入手。

而这样的质数点很多，所以是不是嗅到了

$$ 换根DP $$

这题的主框架就是换根DP

$$dp1[u]=(\sum_{v\in u的节点，且v不是质数} dp1[v]) + (u是质数 ? 0 : 1)$$

而计算的时候，只计算质数的节点

而最难的部分，并不是换根框架，而是路径计算

这里涉及到组合统计优化

- 路径一端是质数
- 路径两端都是非质数
  $$ w是换根时从上往下传递的非质数个数 $$
  $$ res1 = dp1[u] + w $$
  $$ res2 = w * dp1[u] $$
  $$ res2 += \sum_{v\in u的子节点}(dp1[u] + w - dp1[v]) * dp1[v]$$
  $$ res[u] = res1 + res2 / 2 $$
  $$ ans = \sum_{u是质数} res[u] $$

这题难，应该就是难在这里，时间复杂度为$O(N)$

```java
class Solution {

    // 质数表
    static boolean[] isPrime;
    static {
        int n = 10_0000;
        isPrime = new boolean[n + 1];
        boolean[] vis = new boolean[n + 1];
        for (int i = 2; i <= n; i++) {
            if (!vis[i]) {
                isPrime[i] = true;

                if ((long)i * i > n) continue;
                for (int j = i * i ; j <= n; j+=i) {
                    vis[j] = true;
                }
            }
        }
    }

    List<Integer>[]g;
    long[] dp1;
    long[] dp2;

    void dfs1(int u, int fa) {
        dp1[u] = isPrime[u] ? 0 : 1;
        for (int v: g[u]) {
            if (v == fa) continue;
            dfs1(v, u);
            if (!isPrime[v]) {
                dp1[u] += dp1[v];
            }
        }
    }

    long gAns = 0;
    void dfs2(int u, int fa, long w) {

        if (isPrime[u]) {
            // 这是 质数为路径端点 
            gAns += dp1[u] + w;

            // 下面是非质数为路径两端点的情况
            long tmp = 0;
            tmp += w * dp1[u];
            for (int v: g[u]) {
                if (v == fa) continue;
                if (!isPrime[v]) {
                    tmp += (w + dp1[u] - dp1[v]) * dp1[v];
                }
            }

            gAns += tmp/ 2;
        }

        for (int v: g[u]) {
            if (v == fa) continue;
            if (isPrime[u]) {
                // 如果父节点是质数，就传递0
                dfs2(v, u, 0);
            } else {
                // 如果父节点非质数
                long cp = dp1[u] + w;
                if (!isPrime[v]) {
                    cp -= dp1[v];
                }
                dfs2(v, u, cp);
            }
        }
    }

    public long countPaths(int n, int[][] edges) {

        g = new List[n + 1];
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int[] e: edges) {
            g[e[0]].add(e[1]);
            g[e[1]].add(e[0]);
        }

        dp1 = new long[n + 1];

        dfs1(1, -1);

        dp2 = new long[n + 1];
        dfs2(1, -1, 0);

        return gAns;

    }

}
```

### 并查集解法(补充)

从质数点出发，本质上对相邻的非质数连通集合，进行组合计算。

![image.png](https://pic.leetcode.cn/1695548663-VQcSaH-image.png)



```java
class Solution {
    
    static int M = 10_0000;
    static boolean[] isPrime = new boolean[M + 1];
    static {
        Arrays.fill(isPrime, true);
        isPrime[0] = isPrime[1] = false;
        for (int i = 2; i <= M; i++) {
            if (isPrime[i]) {
                if ((long)i * i > M) continue;
                for (int j = i * i; j <= M; j += i) {
                    isPrime[j] = false;
                }
            }
        }
    }
    
    public long countPaths(int n, int[][] edges) {
    
        List<Integer> []g = new List[n + 1];
        Arrays.setAll(g, x -> new ArrayList<>());
        
        // 构建非质数组成的连通子图
        Dsu dsu = new Dsu(n);
        for (int[] e: edges) {
            int u = e[0], v = e[1];
            if (!isPrime[u] && !isPrime[v]) {
                dsu.union(u, v);
            }
            
            g[u].add(v);
            g[v].add(u);
        }
        
        long ans = 0;
        for (int i = 1; i <= n; i++) {
            // 从质数点切入
            if (isPrime[i]) {
                // 增量式求解，这样可以避免分类讨论和除半
                // 很像启发式合并时那种技巧
                long res = 1l;
                for (int v: g[i]) {
                    if (!isPrime[v]) {
                        ans += res * dsu.gSize(v);
                        res += dsu.gSize(v);
                    }
                }
            }
        }
        
        return ans;
        
    }
    
    // 并查集
    static class Dsu {
        int n;
        int[] arr;
        int[] gz;
        
        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
            this.gz = new int[n + 1];
            Arrays.fill(gz, 1);
        }
        
        int find(int u) {
            if (arr[u] == 0) return u;
            return arr[u] = find(arr[u]);
        }
        
        void union(int u, int v) {
            int ai = find(u), bi = find(v);
            if (ai != bi) {
                arr[ai] = bi;
                gz[bi] += gz[ai];
            }
        }
        
        int gSize(int u) {
            return gz[find(u)];
        }
    }
    
}
```

---

# 写在最后

我什么也做不到，但是我会努力成为你心目中的英雄。

![image.png](https://pic.leetcode.cn/1695529313-gNQszN-image.png)
