# 第 361 场周赛 解题报告 | 珂学家 | 思维场 + 同余加权 + 树上前缀和&LCA


---

# 前言

![image.png](https://pic.leetcode.cn/1693722297-bntCeh-image.png)

可莉喜欢毛茸茸的东西。比如嘟嘟可、蒲公英，还有雷泽的头发。

---

# 整体评价


周赛考LCA, 其实我预判到了，思维场挺难的。


---
## [T1. 统计对称整数的数目](https://leetcode.cn/contest/weekly-contest-361/problems/count-symmetric-integers/)

模拟题, 遍历是最好的算法

```java
class Solution {
    public int countSymmetricIntegers(int low, int high) {
        int ans = 0;
        for (int i = low; i <= high; i++) {
            String s = String.valueOf(i);
            if (s.length() % 2 == 0) {
                int v1 = 0, v2 = 0;
                for (int j = 0; j < s.length(); j++) {
                    if (j < s.length() / 2) {
                        v1 = v1 + s.charAt(j) - '0';
                    } else {
                        v2 = v2 + s.charAt(j) - '0';
                    }
                }
                if (v1 == v2) {
                    System.out.println(i);
                    ans++;   
                }
            }
        }
        return ans;
    }
}
```

---
## [T2. 生成特殊数字的最少操作](https://leetcode.cn/problems/minimum-operations-to-make-a-special-number/)

思维题，如果最终的数能被25整除

那末尾一定是 "00", "25", "50", "75"

还有一种特殊情况，去掉所有非0的字符

然后取其最小值

```java
class Solution {

    // 构建末尾为p的最小代价，是字符串匹配问题
    int solve(char[] str, char[] p) {
        int j = 1;
        for (int i = str.length - 1; i >= 0; i--) {
            if (str[i] == p[j]) {
                j--;
            }
            if (j == -1) {
                return (str.length - i) - 2;
            }
        }
        return str.length;
    }


    public int minimumOperations(String num) {
        // 00, 25, 50, 75
        char[] str = num.toCharArray();

        int[] res = new int[] {
                solve(str, "00".toCharArray()),
                solve(str, "25".toCharArray()),
                solve(str, "50".toCharArray()),
                solve(str, "75".toCharArray()),
                num.replaceAll("0", "").length(),  // 非0数字全去掉
        };

        return Arrays.stream(res).min().getAsInt();
    }
    
}
```

----
## [T3. 统计趣味子数组的数目](https://leetcode.cn/problems/count-of-interesting-subarrays/)

同余加权map计数

![image.png](https://pic.leetcode.cn/1693806163-IZczVw-image.png)

还是从图来解释，整个过程

最原始的思路，就是找到最小的区间(其满足的个数 cnt % modulo == k)

那它的延伸区间，就是$span_{left}\times span_{right}$

相当于枚举完所有这样的最小区间，即可求出答案，但是这样的时间复杂度为$O(n^2)$，显然没法接受$(n=10^5)$

有没有办法可以优化

可以观察到 $span_{left}\times span_{right}$ 这个公式

如果固定右端点，其实可以扩展为

$\sum_{i \in 同余S} span_{left_i}\times span_{right}$

如果把$span_{left}$理解为加权

那这题就变成了 同余加权 map，就可以降一维度，时间复杂度为$O(n)$

不过这题难在 $k=0$ 上， 因为它是由非满足点构成，因此需要特判。

因为这个k=0，感觉硬生生做出了hard的感觉来了。

```java
class Solution {
    
    // cnt1 - cnt2 = k + x*m
    public long countInterestingSubarrays(List<Integer> nums, int modulo, int k) {
        // 同余
        int n = nums.size();
        List<Integer> idx = new ArrayList<>();

        for (int i = 0; i < nums.size(); i++) {
            int v = nums.get(i);
            if (v % modulo == k) {
                idx.add(i);
            }
        }

        long ans = 0;
        Map<Integer, Long> hash = new HashMap<>();
        for (int i = 0; i < idx.size(); i++) {
            int no = (i + 1) % modulo;
            
            // 这是加权
            int offset = (i == 0) ? idx.get(i) + 1 : idx.get(i) - idx.get(i - 1);
            hash.merge(no, (long)offset, Long::sum);
            
            // 计算同余点
            int k1 = ((no - (k - 1)) % modulo + modulo) % modulo;
            int span = (i == idx.size() - 1) ? (n - idx.get(i)) : (idx.get(i + 1) - idx.get(i));
            ans += hash.getOrDefault(k1, 0l) * span;
        }
        if (k == 0) {
            // 特别处理
            if (idx.size() > 0) {
                ans += idx.get(0) * (idx.get(0) + 1) / 2;
                for (int i = 0; i < idx.size() - 1; i++) {
                    int span = idx.get(i + 1) - idx.get(i) - 1;
                    ans += (long)span * (span + 1) / 2;
                }
                ans += (long)(n - 1 - idx.get(idx.size() - 1)) * (n - idx.get(idx.size() - 1)) / 2;
            } else {
                ans += (long)n * (n + 1) / 2;    
            }
            
        }
        return ans;                
    }

}

```

---

比赛的时候，其实没想到0-1转换，

如果0-1转换，并构建前缀和模型

如果按照这个思路，那这题就是秒杀了

```java
class Solution {
    public long countInterestingSubarrays(List<Integer> nums, int modulo, int k) {
        Map<Integer, Integer> hash = new HashMap<>();
        hash.put(0, 1);
        int acc = 0;
        
        long ans = 0;
        for (int i = 0; i < nums.size(); i++) {
            int v = nums.get(i) % modulo == k ? 1 : 0;
            acc += v;
            int pv = ((acc - k) % modulo + modulo) % modulo;
            
            ans += hash.getOrDefault(pv, 0);
            hash.merge(acc % modulo, 1, Integer::sum);
        }
        
        return ans;
    }
}
```

---
## [T4. 边权重均等查询](https://leetcode.cn/problems/minimum-edge-weight-equilibrium-queries-in-a-tree/)

树上前缀和 + LCA板子

这个题的切入点，其实是边权的值域为26

因此需要一个自顶向下的边权树上前缀和

定义$path[u][w]$为顶点u, w边权，自顶向下的个数

```java
int[][] path;
List<int[]>[]g;

// 自顶向下的dfs
void dfs(int u, int fa) {
    for (int[] v: g[u]) {
        if (v[0] == fa) continue;

        // 先处理
        for (int i = 1; i <= 26; i++) {
            path[v[0]][i] += path[u][i];
        }
        path[v[0]][v[1]]++;

        // 再dfs子节点
        dfs(v[0], u);
    }
}
```

对于某个查询u, v, 因为最简路径，必然是lca为root

则改动的最小代价为

${ max_{i=1}^{i=26} (depth[u]+depth[v] - 2 * depth[root]) - (path[u][i] + path[v][i] - path[root][i] * 2) }$


> ${depth[u]+depth[v] - 2 * depth[root]}$ 为路径长度

> ${path[u][i] + path[v][i] - path[root][i] * 2}$ 某个边权的个数


```java
class Solution {

    int[][] path;
    List<int[]>[]g;

    void dfs(int u, int fa) {
        for (int[] v: g[u]) {
            if (v[0] == fa) continue;

            for (int i = 1; i <= 26; i++) {
                path[v[0]][i] += path[u][i];
            }
            path[v[0]][v[1]]++;

            dfs(v[0], u);
        }
    }

    public int[] minOperationsQueries(int n, int[][] edges, int[][] queries) {
        path = new int[n][27];

        g = new List[n];
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int[] e: edges) {
            g[e[0]].add(new int[] {e[1], e[2]});
            g[e[1]].add(new int[] {e[0], e[2]});
        }
        // 预处理
        dfs(0, -1);
        this.n = n;

        // 构建lca
        setUpLca();

        // 在线查询
        int ck = queries.length;
        int[] res = new int[ck];
        for (int i = 0; i < queries.length; i++) {
            int u = queries[i][0], v = queries[i][1];
            int root = lca(u, v);
            int mx = depth[u] + depth[v] - depth[root] * 2;
            res[i] = Integer.MAX_VALUE;
            for (int j = 1; j <= 26; j++) {
                res[i] = Math.min(res[i], mx - (path[u][j] + path[v][j] - 2 * path[root][j]));
            }
        }
        return res;
    }

    // lca 板子
    int n, m;
    int[] depth;
    int[][] f;

    public void setUpLca() {
        // 倍增
        m = (int)(Math.log(n) / Math.log(2)) + 1;
        f = new int[n][m + 1];
        depth = new int[n];

        Deque<Integer> deq = new ArrayDeque<>();
        deq.offer(0);
        depth[0] = 1;

        while (!deq.isEmpty()) {
            int now = deq.poll();
            for (int[] e: g[now]) {
                int v = e[0];
                if (depth[v]!=0) continue;
                depth[v] = depth[now] + 1;
                f[v][0] = now;
                for (int i = 1; i <= m; i++) {
                    f[v][i] = f[f[v][i - 1]][i - 1];
                }
                deq.offer(v);
            }
        }
    }

    public int lca(int x, int y) {
        if (depth[x] > depth[y]) {
            // swap
            int t = x;
            x = y;
            y = t;
        }

        // y像x对齐 （高度)
        for (int i = m; i >= 0; i--) {
            if (depth[f[y][i]] >= depth[x]) y = f[y][i];
        }

        if (x == y) return x;

        for (int i = m; i >= 0; i--) {
            if (f[y][i] != f[x][i]) {
                y = f[y][i];
                x = f[x][i];
            }
        }
        return f[x][0];
    }

}
```


---

# 写在最后

唉，你是好人，我是坏孩子.等我这次的禁闭结束、好好反省过以后，再来找你带我出去玩……

![image.png](https://pic.leetcode.cn/1693722408-GmWbZa-image.png)
