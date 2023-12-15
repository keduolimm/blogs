# 第 117 场双周赛 解题报告 | 珂学家 | 状压DP + 多路归并


---

# 前言

![image.png](https://pic.leetcode.cn/1699718980-rBPHRW-image.png)


---

# 整体评价

整体感觉不难，t4是一道很基础的多路归并题，t3是个状压题，需要设计好状态，以及维护好转移状态。

---

## [T1. 给小朋友们分糖果 I](https://leetcode.cn/contest/biweekly-contest-117/problems/distribute-candies-among-children-i/)

和T2一起讲

---

## [T2. 给小朋友们分糖果 II](https://leetcode.cn/contest/biweekly-contest-117/problems/distribute-candies-among-children-ii/)

这题可以枚举1个小朋友的糖果，然后快速评估剩下糖果分给2个小朋友的方案数

所以这题的难点在于

$$给定x个糖果，2个小朋友分的方案总数$$

那这个咋计算呢？

可以引入一个小朋友分糖果上下界

$$ lower = max(0, x - limit) $$

$$ upper = min(x, limit) $$

那么只有当 ${ lower \le upper }$才有解

$$ upper - lower + 1 $$

```java
class Solution {
    public long distributeCandies(int n, int limit) {
        long acc = 0;
        for (int i = 0; i <= Math.min(n, limit); i++) {
            int x = n - i;
            int l = Math.max(x - limit, 0);
            int r = Math.min(x, limit);
            if (l <= r) {
                acc += r - l + 1;
            }
        }
        return acc;
    }
}
```

---

## [T3. 重新排列后包含指定子字符串的字符串数目](https://leetcode.cn/contest/biweekly-contest-117/problems/number-of-strings-which-can-be-rearranged-to-contain-substring/)


一眼状压，难点在于

$$如何维护2e，这个状态如何设计$$

${让字母'l', 't', 对应二进制的2^0, 2^1}$

${让字母'e', 对应二进制的2^2, 如果有2个’e', 则对应 2^3 + 2^2}$

其他字母视为 0，只是权重为23。

这样合法的状态数为

$$ dp[15] = dp[0b1111]$$

状态转移

$$dp[i][s | x)] <= dp[i - 1][s], x\in ('l', 't') $$

$$dp[i][s | 4)] <= dp[i - 1][s], x = 'e'， s不包含‘e’ $$

$$dp[i][s | 12)] <= dp[i - 1][s], x = 'e'， s包含‘e’ $$

$$dp[i][s] <= dp[i - 1][s] \times 23, x为其他字母 $$

状压的设计方案蛮多的。

这边的时间复杂度为$O(n * 16 * 4)=O(64n)$


```java
class Solution {

    static int SZ = 10_0000;
    static long mod = 10_0000_0007l;
    static long[] res = new long[SZ + 1];

    // 静态初始化
    static {
        
        long[] dp = new long[16];
        for (int i = 0; i < 3; i++) {
            dp[1 << i]++;
        }
        dp[0] = 23;

        res[1] = 0;
        for (int i = 2; i <= SZ; i++) {
            long[] nextDp = new long[16];
            for (int k = 0; k < 16; k++) {
                for (int j = 0; j < 3; j++) {
                    if (j == 2 && (k & (1 << j)) != 0) {
                        nextDp[k | 8] = (nextDp[k | 8] + dp[k]) % mod;
                    } else {
                        nextDp[k | (1 << j)] = (nextDp[k | (1 << j)] + dp[k]) % mod;
                    }
                }
                nextDp[k] = (nextDp[k] + dp[k] * 23 % mod) % mod;
            }
            dp = nextDp;
            res[i] = dp[15];
        }
    }


    public int stringCount(int n) {
        return (int)res[n];
    }

}
```

----

## [T4. 购买物品的最大开销](https://leetcode.cn/contest/biweekly-contest-117/problems/maximum-spending-after-buying-items/)

多路归并的板子题,

因为题目追求的是最大的结果，所以把价值大的放在最后，这样和天数的乘积就越大。

引入优先队列(最小堆)，里面的项为

$$(row, col, value),  行号，行中的列索引，对一个的值$$

- 每次挑选当前价值最少的

- 然后把那一行的下一个元素加入到优先队列即可


```java
class Solution {

    public long maxSpending(int[][] values) {
        PriorityQueue<int[]> pp = new PriorityQueue<>(Comparator.comparing(x -> x[2]));

        int m = values.length;
        int n = values[0].length;
        for (int i = 0; i < m; i++) {
            pp.offer(new int[] {i, n - 1, values[i][n - 1]});
        }

        int day = 0;
        long acc = 0;
        while (!pp.isEmpty()) {
            int[] cur = pp.poll();
            day++;
            acc += (long)day * values[cur[0]][cur[1]];

            if (cur[1] > 0) {
                pp.offer(new int[] {cur[0], cur[1] - 1, values[cur[0]][cur[1] - 1]});
            }
        }

        return acc;
    }

}
```

----

# 写在最后

![image.png](https://pic.leetcode.cn/1699718923-ybqxzz-image.png)

