

# 第 125 场双周赛 解题报告 | 珂学家 | 树形DP + 组合数学

---

# 前言

![image.png](https://pic.leetcode.cn/1709447875-wSPVhi-image.png)


---

# 整体评价

T4感觉有简单的方法，无奈树形DP一条路上走到黑了，这场还是有难度的。

---

## [T1. 超过阈值的最少操作数 I](https://leetcode.cn/contest/biweekly-contest-125/problems/minimum-operations-to-exceed-threshold-value-i/)


思路: 模拟

```java []
class Solution {
    public int minOperations(int[] nums, int k) {
        return (int)Arrays.stream(nums).filter(x -> x < k).count();
    }
}
```

---
## [T2. 超过阈值的最少操作数 II](https://leetcode.cn/contest/biweekly-contest-125/problems/minimum-operations-to-exceed-threshold-value-ii/)


思路: 模拟

按题意模拟即可，需要注意int溢出问题就行。

```java []
class Solution {
    public int minOperations(int[] nums, int k) {
        PriorityQueue<Long> pq = new PriorityQueue<>();
        for (long num : nums) {
            pq.offer(num);
        }
        int ans = 0;
        while (pq.peek() < k) {
            long x = pq.poll(), y = pq.poll();
            pq.offer(Math.min(x, y) * 2 + Math.max(x, y));
            ans++;
        }
        return ans;
    }
}
```

---
## [T3. 在带权树网络中统计可连接服务器对数目](https://leetcode.cn/contest/biweekly-contest-125/problems/count-pairs-of-connectable-servers-in-a-weighted-tree-network/)

思路: 枚举 + dfs/bfs + 组合数学

因为树的点数n($n\le1000$), 所以枚举点，从该点进行dfs/bfs，然后对每个分支进行组合统计。

组合统计的核心

$$点u出发的各个分支满足整除的数，组合序列为 c_0, c_1, c_2, c_3, ..., c_m$$

$$ 其 s = \sum_{i=0}^{i=m} c_i$$

$$ 结果为 res[u] = (\sum_{i=0}^{i=m} c_i * (s - c_i)) / 2$$

这样时间复杂度为 $O(n^2)$, 是可以接受的。

```java []
class Solution {

    List<int[]> []g;
    int signalSpeed;
    
    // fa是阻断点
    int bfs(int u, int w, int fa) {
        int res = 0;
        boolean[] vis = new boolean[g.length];
        Deque<int[]> deq = new ArrayDeque<>();
        vis[u] = true;
        deq.offer(new int[] {u, w});
        if (w % signalSpeed == 0) {
            res ++;
        }
        while (!deq.isEmpty()) {
            int[] cur = deq.poll();
            int p = cur[0];
            int v = cur[1];
            for (int[] e: g[p]) {
                if (e[0] == fa) continue;
                if (vis[e[0]]) continue;
                if ((v + e[1]) % signalSpeed == 0) res++;
                deq.offer(new int[] {e[0], v + e[1]});
                vis[e[0]] = true;
             }
        }
        return res;
    }

    public int[] countPairsOfConnectableServers(int[][] edges, int signalSpeed) {
        int n = edges.length + 1;

        g = new List[n];
        Arrays.setAll(g, x->new ArrayList<>());
        this.signalSpeed = signalSpeed;
        
        for (int[] e: edges) {
            g[e[0]].add(new int[] {e[1], e[2]});
            g[e[1]].add(new int[] {e[0], e[2]});
        }

        int[] res = new int[n];
        for (int i = 0; i < n; i++) {
            int sum = 0;
            
            List<Integer> lists = new ArrayList<>();
            for (int[] e: g[i]) {
                int tmp = bfs(e[0], e[1], i);
                lists.add(tmp);
                sum += tmp;
            }

            int tot = 0;
            for (int j = 0; j < lists.size(); j++) {
                tot += lists.get(j) * (sum - lists.get(j));
            }
            res[i] = tot / 2;
        }
        return res;
    }

}
```

---

如果该题把n变成$n\le10^5$, 那该如何解呢？

$$感觉换根DP可行，但是需要限制 signalSpeed 范围在100之内,  这样可控制在O(10^7)$$

如果signalSpeed很大，感觉没辙啊。

---

## [T4. 最大节点价值之和](https://leetcode.cn/contest/biweekly-contest-125/problems/find-the-maximum-sum-of-node-values/)

思路: 树形DP

树形DP的解法更加具有通用性，所以比赛就沿着这个思路写。

$$如果操作不是异或，那这个思路就更有意义$$

对于任意点u, 其具备两个状态。

$$dp[u][0], dp[u][1], 表示参与和没有参与异或下的以u为根节点的子树最大和。$$

那么其转移方程，其体现在当前节点u和其子节点集合S($v\in u的子节点$)的迭代递推转移。

```java []
class Solution {

    int k;
    int[] nums;
    List<Integer>[]g;

    long[][] dp;

    void dfs(int u, int fa) {
        // 该节点没参与， 该节点参与了
        long r0 = nums[u], r1 = Long.MIN_VALUE / 10;

        for (int v: g[u]) {
            if (v == fa) continue;
            dfs(v, u);

            long uRev0 = r0 + (nums[u]^k) - nums[u];
            long uRev1 = r1 - (nums[u]^k) + nums[u];
            long vRev0 = dp[v][0] + (nums[v]^k) - nums[v];
            long vRev1 = dp[v][1] - (nums[v]^k) + nums[v];

            long x0 = Math.max(
                    r0 + Math.max(dp[v][0], dp[v][1]),
                    Math.max(uRev1 + vRev1, uRev1 + vRev0)
            );

            long x1 = Math.max(
                    r1 + Math.max(dp[v][1], dp[v][0]),
                    Math.max(uRev0 + vRev0, uRev0 + vRev1)
            );

            r0 = x0;
            r1 = x1;
        }

        dp[u][0] = r0;
        dp[u][1] = r1;
    }

    public long maximumValueSum(int[] nums, int k, int[][] edges) {
        int n = nums.length;

        this.g = new List[n];
        this.nums = nums;
        this.k = k;

        this.dp = new long[n][2];
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int[] e: edges) {
            g[e[0]].add(e[1]);
            g[e[1]].add(e[0]);
        }

        dfs(0, -1);
        return Math.max(dp[0][0], dp[0][1]);
    }
}
```

```python3 []
class Solution:
    def maximumValueSum(self, nums: List[int], k: int, edges: List[List[int]]) -> int:
        n = len(nums)
        g = [[] for _ in range(n)]
        for e in edges:
            g[e[0]].append(e[1])
            g[e[1]].append(e[0])
        
        dp = [[0] * 2 for _ in range(n)]
        
        def dfs(u, fa):
            r0, r1 = nums[u], -0x3f3f3f3f3f
            
            for v in g[u]:
                if v == fa:
                    continue
                dfs(v, u)
                
                uRev0 = r0 + (nums[u]^k) - nums[u];
                uRev1 = r1 - (nums[u]^k) + nums[u];
                vRev0 = dp[v][0] + (nums[v]^k) - nums[v];
                vRev1 = dp[v][1] - (nums[v]^k) + nums[v];
                
                t0 = max(r0 + max(dp[v][0], dp[v][1]), max(uRev1 + vRev0, uRev1 + vRev1))
                t1 = max(r1 + max(dp[v][0], dp[v][1]), max(uRev0 + vRev0, uRev0 + vRev1))
                
                r0, r1 = t0, t1
            
            dp[u][0], dp[u][1] = r0, r1
        
        dfs(0, -1)
        
        return max(dp[0][0], dp[0][1])
```

---

由于异或的特点，所以这题可以抛弃边的束缚。

$$任意两点u,v，可以简单构造一条路径，只有端点(u,v)出现1次，其他点都出现2次$$


异或涉及边的两点，因此异或的点必然是偶数个，这是唯一的限制.



```java []
class Solution {
    public long maximumValueSum(int[] nums, int k, int[][] edges) {
        long sum = 0;
        PriorityQueue<Long> pq = new PriorityQueue<>(Comparator.comparing(x -> -x));
        for (int v: nums) {
            pq.offer((long)(v ^ k) - v);
            sum += v;
        }
        
        while (pq.size() >= 2) {
            long t1 = pq.poll();
            long t2 = pq.poll();
            if (t1 + t2 > 0) {
                sum += t1 + t2;
            } else {
                break;
            }
        }
        
        return sum;
    }
}
```

```python3 []
class Solution:
    def maximumValueSum(self, nums: List[int], k: int, edges: List[List[int]]) -> int:
        s = sum(nums)
        arr = [(v ^ k) - v for v in nums]
        arr.sort(key=lambda x: -x)
        
        n = len(nums)
        for i in range(0, n, 2):
            if i + 1 < n and arr[i] + arr[i + 1] > 0:
                s += arr[i] + arr[i + 1]
        return s
```


---

# 写在最后



![image.png](https://pic.leetcode.cn/1709447912-vMRgau-image.png)
