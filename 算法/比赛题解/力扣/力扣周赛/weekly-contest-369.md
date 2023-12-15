# 第 369 场周赛 解题报告 | 珂学家 | 状态机 DP + 树形 DP


---
# 前言


![image.png](https://pic.leetcode.cn/1698576687-GytqwA-image.png)

命运的红线一旦断了，就再也接不上了。

---
# 整体评价

T3是道经典的状态机DP， T4是一道自顶向下的树形DP，它需要把祖先的第二类操作次数往下传递，而且这里存在一个大剪枝，或者说状态缩点。


---

## [A. 找出数组中的 K-or 值](https://leetcode.cn/contest/weekly-contest-369/problems/find-the-k-or-of-an-array/)

题目有些绕，模拟即可。

其实就是 第i位为1的个数，大于等于k，即有效，也就是二进制题中特有的拆位思路。

```java
class Solution {
    public int findKOr(int[] nums, int k) {
        int res = 0;
        for (int i = 0; i < 31; i++) {
            int tmp = 0;
            for (int v: nums) {
                if ((v & (1 << i)) != 0) {
                    tmp++;
                }
            }
            if (tmp >= k) {
                res |= (1 << i);
            }
        }
        return res;
    }
}
```

---

## [B. 数组的最小相等和](https://leetcode.cn/contest/weekly-contest-369/problems/minimum-equal-sum-of-two-arrays-after-replacing-zeros/)

很好的一道思维题，考察有效范围的上下界

假设数组$arr其和为s, 0的个数为m$

那么
$$有效下界lower=s+m$$
$$
有限上界upper = \left \{\begin{matrix}
s, m = 0  \\  
无穷大, m > 0
\end{matrix}\right.
$$

如果数组$arr, brr$的上下界不存在交集，则无解

如果存在交集，那最小值为

$$ max(s1+m1, s2 + m2) $$


```java
class Solution {
    public long minSum(int[] nums1, int[] nums2) {
    
        long s1 = 0, s2 = 0;
        int m1 = 0, m2 = 0;
        for (int v: nums1) {
            s1 += v;
            if (v == 0) m1++;
        }
        
        for (int v: nums2) {
            s2 += v;
            if (v == 0) m2++;
        }
        
        long lower1 = m1 + s1;
        long lower2 = m2 + s2;
        
        long inf = Long.MAX_VALUE / 10;
        long upper1 = m1 > 0 ? inf : s1;
        long upper2 = m2 > 0 ? inf : s2;
        
        
        if (lower1 > upper2 || lower2 > upper1) return -1;
        
        return Math.max(lower1, lower2);
        
    }
    
}
```

---

## [C. 使数组变美的最小增量运算数](https://leetcode.cn/contest/weekly-contest-369/problems/minimum-increment-operations-to-make-array-beautiful/)

一道经典的状态机DP

引入3个状态，$s0, s1, s2$, 这三个节点都是有效值

- $s0$代表当前节点值$\ge k$的最小代价
- $s1$代表前1个节点值$\ge k$的最小代价
- $s2$代表前2个节点值$\ge k$的最小代价

线性递推下，其转移方程为

$$ s0_i = min(s0_{i-1}, s1_{i-1}, s2_{i-1}) + max(0, k - arr[i])  $$
$$ s1_i = s0_{i - 1}  $$
$$ s2_i = s1_{i - 1}  $$

最后的结果为

$$ min(s0_{n-1}, s1_{n-1}, s2_{n-1}) $$

因为是线性的，可以压缩空间为3个变量

```java
class Solution {
    
    public long minIncrementOperations(int[] nums, int k) {
        long inf = Long.MAX_VALUE / 10;
        
        long s0 = inf, s1 = 0, s2 = 0;
        s0 = Math.max(0, k - nums[0]);
        
        for (int i = 1; i < nums.length; i++) {
        
            long t0 = Math.min(Math.min(s0, s1), s2) + Math.max(0, k - nums[i]);
            long t1 = s0;
            long t2 = s1;
            
            
            s0 = t0;
            s1 = t1;
            s2 = t2;
        }
        
        return Math.min(Math.min(s0, s1), s2);
    }
    
}
```

---
## [D. 收集所有金币可获得的最大积分](https://leetcode.cn/contest/weekly-contest-369/problems/maximum-points-after-collecting-coins-from-all-nodes/)


因为依赖是由上往下传递的，所以这题是自顶向下的树形DP

令$opt[u][w], 表示第u个节点，其祖先减半了w次$

$而opt[u][w]本身选择第一类操作，和第二类操作$

其状态转移：

$$ t1 = half(coins[u], w) - k + \sum_{v\in u的子节点} opt[v][w]  $$
$$ t2 = half(coins[u], w + 1) + \sum_{v\in u的子节点} opt[v][w + 1]$$

$$ opt[u][w] = max(t1, t2)$$

但是这里面有个需要注意的地方，即减半操作和深度有关，会导致状态膨胀。

幸好值最大为$10^4$, 最多除以14次。

所以这里的trick点

$$就是超过14次，其子树最大的价值一定为0$$

这样状态数，就控制在$O(n*15)$

总结一下，关键的两点

- 自顶向下
- 超过14次减半，缩点归0

```java
class Solution {

    static long inf = -Long.MAX_VALUE / 10;
    List<Integer>[]g;
    int n, k;
    int[] coins;

    Integer[][] memo;

    int dfs(int u, int fa, int odd) {

        if (odd >= 14) return 0;
        
        if (memo[u][odd] != null) {
            return memo[u][odd];
        }

        int now = coins[u];
        for (int i = 0; i < odd; i++) {
            now /= 2;
        }

        int t1 = now - k;
        for (int v: g[u]) {
            if (v == fa) continue;
            t1 += dfs(v, u, odd);
        }

        int t2 = now / 2;
        for (int v: g[u]) {
            if (v == fa) continue;
            t2 += dfs(v, u, odd + 1);
        }

        return memo[u][odd] = Math.max(t1, t2);

    }

    public int maximumPoints(int[][] edges, int[] coins, int k) {

        this.n = coins.length;
        g = new List[n];
        Arrays.setAll(g, x -> new ArrayList<>());
        this.k = k;
        this.coins = coins;

        for (int[] e: edges) {
            g[e[0]].add(e[1]);
            g[e[1]].add(e[0]);
        }
        memo = new Integer[n][15];

        return dfs(0, -1, 0);
    }

}
```


---

# 写在最后

心若能新生于人世，夜半之月也会眷恋吗?

![image.png](https://pic.leetcode.cn/1698576845-UYNslH-image.png)
