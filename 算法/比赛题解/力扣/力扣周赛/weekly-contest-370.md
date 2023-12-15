# 第 370 场周赛｜解题报告 | 珂学家 | 树形DP + RMQ优化的DP


---

# 前言

![image.png](https://pic.leetcode.cn/1699158234-yZOlOe-image.png)


---

# 整体评价

T4很经典，有点板。T3, 我感觉题目描述有点问题，题面和case 2冲突了，泪目。理清就很简单了。

---
## [T1. 找到冠军 I](https://leetcode.cn/contest/weekly-contest-370/problems/find-champion-i/)

和T2,一起讲

---

## [T2. 找到冠军 II](https://leetcode.cn/contest/weekly-contest-370/problems/find-champion-ii/)

和T1的差别在于，边维护上，一个是邻接矩阵，一个是邻接边

伪图论题，其实冠军就是没有输过的队伍。

如果把输赢当做出入度计数，

那最终的赢家就是入度为0，冠军则是只有一个入度为0的情况存在时，才存在.

所以打比赛，最完美的策略，就是

$$不打任何一场比赛，然后捡冠军$$

```java
class Solution {
    
    public int findChampion(int n, int[][] edges) {
        // 计算入度
        int[] deg = new int[n];
        for (int[] e: edges) {
            deg[e[1]]++;
        }
        
        int idx = -1;
        int cnt = 0;
        for (int i = 0; i < n; i++) {
            if (deg[i] == 0) {
                cnt++;
                idx = i;
            }
        }
        
        return cnt == 1 ? idx : -1;
    }
    
}
```


---

## [T3. 在树上执行操作以后得到的最大分数](https://leetcode.cn/contest/weekly-contest-370/problems/maximum-score-after-applying-operations-on-a-tree/)

树形DP，这题题目描述和case 2描述冲突了，泪目。

为每个节点，引入两个状态0, 1，表示从当前节点到叶节点存在0，不存在0，其值为最大收益。

因此状态转移为

$$ t1 = \sum_{v\in u的子节点} opt[v][0], u节点不选择$$

$$ t2 = \sum_{v\in u的子节点} pt[v][1] + values[u]， u节点选择$$

$$

opt[u][1] =
\left\{\begin{matrix}

max(t1, t2)&, u不是叶节点 \\

t1&, u是叶节点 \\

\end{matrix}\right.
$$


$$ opt[u][0] = (\sum_{v\in u的子节点} opt[v][0]) + values[u]$$


```java
class Solution {

    int n;
    List<Integer> []g;
    int[] values;

    long[][] dp;
    void dfs(int u, int fa) {

        int cnt = 0;
        long r0 = values[u], r1 = 0;
        long r2 = values[u];
        for (int v: g[u]) {
            if (v == fa) continue;
            dfs(v, u);
            r0 += dp[v][0];
            r1 += dp[v][0];
            r2 += dp[v][1];
            cnt++;
        }

        if (cnt == 0) {
            dp[u][0] = values[u];
            dp[u][1] = 0;
            return;
        }

        dp[u][0] = r0;
        dp[u][1] = Math.max(r1, r2);
    }

    public long maximumScoreAfterOperations(int[][] edges, int[] values) {

        this.n = values.length;

        g = new List[n];
        Arrays.setAll(g, x-> new ArrayList<>());

        for (int[] e: edges) {
            g[e[0]].add(e[1]);
            g[e[1]].add(e[0]);
        }

        this.values = values;

        dp = new long[n][2];
        dfs(0, -1);

        return dp[0][1];
    }

}
```


---

## [T4. 平衡子序列的最大和](https://leetcode.cn/contest/weekly-contest-370/problems/maximum-balanced-subsequence-sum/)

RMQ优化的DP

移项交换

令$f(i) = arr[i] - i$

那这题，就转换为 严格非递增 的序列 最大和

$$ 令g[i]是以i项结尾的最大序列和$$

$$ t1 = \max_{j<i 且 f[i] \le f[j]} g[j]$$

$$ g[i] = max(t1, 0) + arr[i]$$

因为n数据范围为$O(10^5)$, 所以这题需要借助

支持RMQ的数据结构来优化

可以使用线段树，当然这边可以偷懒使用树状数组

这边的树状数组很特殊，它只支持[0, l]的最大值查询

```java
class Solution {

    static long inf = Long.MIN_VALUE / 10;

    public class BIT {
        int n;
        long[] arr;
        public BIT(int n) {
            this.n = n;
            this.arr = new long[n + 1];
            Arrays.fill(arr, inf);
        }

        void update(int p, long d) {
            while (p <= n) {
                arr[p] = Math.max(arr[p], d);
                p += p &-p;
            }
        }
        long query(int p) {
            long res = 0;
            while (p > 0) {
                res = Math.max(arr[p], res);
                p -= p & -p;
            }
            return res;
        }
    }


    public long maxBalancedSubsequenceSum(int[] nums) {

        int n = nums.length;
        long[] f = new long[n];
        for (int i = 0; i < n; i++) {
            f[i] = nums[i] - i;
        }

        // 这边需要离散化
        Set<Long> ts = new TreeSet<>();
        for (long v: f) {
            ts.add(v);
        }
        int ptr = 0;

        Map<Long, Integer> idMap = new HashMap<>();
        for (var k: ts) {
            idMap.put(k, ++ptr);
        }

        long ans = inf;
        BIT bit = new BIT(ptr);
        for (int i = 0; i < n; i++) {
            long v = f[i];
            int idx = idMap.get(v);

            long r = bit.query(idx);
            long x = Math.max(r, 0) + nums[i];

            bit.update(idx, x);
            ans = Math.max(ans, x);
        }

        return ans;

    }

}
```

---
# 写在最后


![image.png](https://pic.leetcode.cn/1699158291-GcFWvR-image.png)
