# 第 126 场双周赛 解题报告 | 珂学家 | 贡献法思维场 + 贪心构造 + 0-1背包

----

# 前言

![image.png](https://pic.leetcode.cn/1710605287-PnZAAV-image.png)



---


# 整体评价

T3是道好题，一开始思路偏了往按字母前缀和和DP去想了，但是感觉很难下手，后来发现从贡献的角度，其实和位子无关系，只需要贪心即可。

T4也是一道贡献思路题，理清核心的点，就能简单构建出0-1背包，求得总共方案数(不涉及去重)。

----

## [T1. 求出加密整数的和](https://leetcode.cn/contest/biweekly-contest-126/problems/find-the-sum-of-encrypted-integers/)

思路: 模拟题

java的Math.max不支持char类型重载，导致写起来很变扭, T_T.

```java []
class Solution {
    public int sumOfEncryptedInt(int[] nums) {
        int s = 0;
        for (int v: nums) {
            String sv = String.valueOf(v);
            int m = sv.charAt(0) - '0';
            for (char c: sv.toCharArray()) {
                m = Math.max(m, c - '0');
            }
            String mv = "" + (char)(m + '0');
            s += Integer.valueOf(mv.repeat(sv.length()));
        }
        return s;
    }
}
```

---

## [T2. 执行操作标记数组中的元素](https://leetcode.cn/contest/biweekly-contest-126/problems/mark-elements-on-array-by-performing-queries/)

思路: 模拟题

用优先队列来实现

```java []
class Solution {
    public long[] unmarkedSumArray(int[] nums, int[][] queries) {
        PriorityQueue<int[]> pp = new PriorityQueue<>(
                Comparator.comparingInt((int[] a) -> a[0]).thenComparingInt(a -> a[1])
        );

        int n = nums.length;
        boolean[] vis = new boolean[n];
        long s = 0;
        for (int i = 0; i < nums.length; i++) {
            s += nums[i];
            pp.offer(new int[] {nums[i], i});
        }

        List<Long> res = new ArrayList<>();
        for (int[] e: queries) {
            int idx = e[0];
            if (!vis[idx]) {
                vis[idx] = true;
                s -= nums[idx];
            }
            for (int i = 0; !pp.isEmpty() && i < e[1]; i++) {
                while (!pp.isEmpty()) {
                    int[] cur = pp.poll();
                    if (!vis[cur[1]]) {
                        vis[cur[1]] = true;
                        s -= nums[cur[1]];
                        break;
                    }
                }
            }
            res.add(s);
        }
        return res.stream().mapToLong(Long::valueOf).toArray();
    }
}
```

---

## [T3. 替换字符串中的问号使分数最小](https://leetcode.cn/contest/biweekly-contest-126/problems/replace-question-marks-in-string-to-minimize-its-value/)

思路: 贪心构造

蛮思维的一道构造题

可以发现，额外添加一个字母，放在任意'?'的位子, 其贡献都是一样的，即位子无关。

那先回到题目的诉求

- 能量值最小（优先）
- 字典序最小

能量值最小，那如何求呢？

$$可以从增量的角度出发，当前那个字母数量少(字母序小)，就选它$$

这样构造出的字符串一定是能量最小的。

同时位子无关，但是字典序最小，所以要排序，把添加的字母按大小依次填充'?'的位子。

```java []
class Solution {
    public String minimizeStringValue(String s) {
        char[] str = s.toCharArray();
        int n = str.length;

        // 贪心构造
        int[] hash = new int[26];
        List<Integer> arr = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (str[i] == '?') {
                arr.add(i);
            } else {
                hash[str[i] - 'a']++;
            }
        }

        PriorityQueue<long[]> pp = new PriorityQueue<>(
            Comparator.comparing((long[] x) -> x[0]).thenComparing(x -> x[1])
        );
        for (int i = 0; i < 26; i++) {
            pp.offer(new long[] {hash[i], i});
        }

        List<Character> res = new ArrayList<>();
        for (int i = 0;i < arr.size(); i++) {
            long[] cur = pp.poll();
            res.add((char)((int)cur[1] + 'a'));
            pp.offer(new long[] {cur[0] + 1l, cur[1]});
        }
        Collections.sort(res);

        for (int i = 0; i < arr.size(); i++) {
            str[arr.get(i)] = res.get(i);
        }
        return new String(str);
    }
}
```



----

## [T4. 求出所有子序列的能量和](https://leetcode.cn/contest/biweekly-contest-126/problems/find-the-sum-of-the-power-of-all-subsequences/)


思路: 贡献法 + DP

这题很典, 涉及2个知识点

- 贡献法

$$从子序列seq(和为k)，其元素个数为m, 倒推其包含的子序列个数为 2^{n - m}$$

所以难点在于

$$维护构建k的序列个数信息，以及求出方案数$$

- DP

其实有点像0-1背包，但又不完全像


$$令dp[i][s][t]为前i项中，共s个元素参与组成和为t的总方案数$$

则其转移方程为


$$dp[i][s][t] = dp[i - 1][s][t] + dp[i - 1][s - 1][t - arr[i]]$$

因为这里面的i是线性递归的，所以可以滚动数组优化。


其实还可以借助0-1背包的逆序遍历技巧，把滚动数组也优化掉。

这样时间复杂都为 $O(n*k^2)$, 空间复杂度为$O(k^2)$

```java[]
class Solution {

    static long mod = (long)1e9 + 7;

    public int sumOfPower(int[] nums, int k) {
        long res = 0;
        int n = nums.length;
        long[] pow2 = new long[n + 1];
        pow2[0] = 1;
        for (int i = 1; i <= n; i++) {
            pow2[i] = pow2[i - 1] * 2 % mod;
        }

        long[][] dp = new long[k + 1][k + 1];
        dp[0][0] = 1;
        for (int i = 0; i < n; i++) {
            int v = nums[i];

            for (int j = 0; j <= k; j++) {
                dp[j][k] = 0;
            }
            for (int j = k - 1; j >= 0; j--) {
                for (int t = k - v; t >= 0; t--) {
                    dp[j + 1][t + v] = (dp[j + 1][t + v] + dp[j][t]) % mod;
                }
            }
            for (int j = 1; j <= k; j++) {
                res += dp[j][k] * pow2[Math.max(0, n - j)] % mod;
                res %= mod;
            }
        }
        return (int)res;
    }

}
```

---

# 写在最后

![image.png](https://pic.leetcode.cn/1710605272-pnPLoh-image.png)
