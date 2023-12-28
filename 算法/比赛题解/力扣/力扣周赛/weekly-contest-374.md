
# 第 374 场周赛 解题报告 | 珂学家 | 拆位前缀和优化+分组滑窗+组合数学

---
# 前言


---
# 整体评价

这场挺难的，2题手速快的话，也能排一个好的名次。

T3是道经典的题，可以借助拆位前缀和来优化，不过整体的时间复杂度也算蛮高了，好像卡c++的常数了。

T4的组合数学好像超纲了，不过力扣周赛是考过几回了，属于常规超纲知识点。

---

## [T1. 找出峰值](https://leetcode.cn/contest/weekly-contest-374/problems/find-the-peaks/)

```java
class Solution {
    public List<Integer> findPeaks(int[] mountain) {
        List<Integer> res = new ArrayList<>();
        for (int i = 1; i < mountain.length - 1; i++) {
            if (mountain[i] > mountain[i - 1] && mountain[i] > mountain[i + 1]) res.add(i);
        }
        return res;
    }
}
``` 

---

## [T2. 需要添加的硬币的最小数量](https://leetcode.cn/contest/weekly-contest-374/problems/minimum-number-of-coins-to-be-added/)

这题是结论题：

$$前缀和大于等于某个数，可以吸纳于集合中，并能自由组合构建1~N的数$$

```java
class Solution {
    public int minimumAddedCoins(int[] coins, int target) {        
        Arrays.sort(coins);
        
        int res = 0;
        int s = 0;
        
        for (int i = 0; s < target && i < coins.length; i++) {
            while (s < target && s + 1 < coins[i]) {
                s += s + 1;
                res++;
            }
            s += coins[i];
        }
        
        while (s < target) {
            s += s + 1;
            res++;
        }
        
        return res;
    }
}
```

---

## [T3. 统计完全子字符串](https://leetcode.cn/contest/weekly-contest-374/problems/count-complete-substrings/)

思路: 拆位+前缀和

时间复杂度相对有些高, $O(26^2 * n)$


```java
class Solution {

    public int countCompleteSubstrings(String word, int k) {
        char[] str = word.toCharArray();

        int n = str.length;

        int[] valid = new int[n + 1];
        int[][] pre = new int[26][n + 1];
        for (int i = 0; i < n; i++) {
            int p = str[i] - 'a';
            for (int j = 0; j < 26; j++) {
                pre[j][i + 1] = pre[j][i] + (p == j ? 1 : 0);
            }

            if (i + 1 < n) {
                int t = Math.abs(str[i] - str[i + 1]);
                valid[i + 1] = valid[i] + (t > 2 ? 1 : 0);
            }
        }

        int res = 0;
        for (int i = 0; i < n; i++) {

            for (int j = i - k + 1; j >= 0; j -= k) {

                int s = valid[i] - valid[j];
                if (s == 0) {

                    int t0 = 0, t1 = 0;
                    for (int t = 0; t < 26; t++) {
                        int x = pre[t][i + 1] - pre[t][j];
                        if (x > 0 && x < k) t0++;
                        else if (x > k) {
                            t1++;
                            break;
                        }
                    }

                    if (t1 > 0) {
                        break;
                    } else if (t0 == 0) {
                        res++;
                    }
                } else {
                    break;
                }

            }

        }
        return res;
    }
}
```

---

## [T4. 统计感冒序列的数目](https://leetcode.cn/contest/weekly-contest-374/problems/count-the-number-of-infection-sequences/)

思路: 组合数学

如果一时找不到规律，按照灵神的建议

$$可以从特殊到一般$$

对于一个区间(长度为m)
- 内部的选择是$2^(m-1)$
- 最左侧/右侧为1

而从全局来看

区间和区间之间，它满足如下公式

$C(n_1, n_1) * C(n_1 + n_2, n_2) * ... * C(n_1+n_2+...+n_t, n_t)$

最终的结果即乘法原理

$$内部和全局累乘即可$$

剩下的就是组合计算的板子题

对于n在$(10^6)$范围内，p很大(质数)，一般采用预处理

这样的话，整个复杂度为$O(n)$

```java

class Solution {
    
    static long mod = (long)1e9 + 7l;
    static int SZ = 10_0000;
    static long[] fac = new long[SZ + 1];
    static long[] inv = new long[SZ + 1];
    
    static {
        fac[0] = 1;
        for (int i = 1; i <= SZ; i++) {
            fac[i] = fac[i - 1] * i % mod;
        }
        
        // 费马小定理
        inv[SZ] = ksm(fac[SZ], mod - 2);
        
        for (int j = SZ - 1; j >= 0; j--) {
            inv[j] = inv[j + 1] * (j + 1) % mod;
        }
    }
    
    // *) 
    static long ksm(long b, long v) {
        long r = 1;
        while (v > 0) {
            if (v % 2 == 1) {
                r = r * b % mod;
            }
            v /= 2;
            b = b * b % mod;
        }
        return r;
    }
    
    long comb(int n, int k) {
        return fac[n] * inv[n - k] % mod * inv[k] % mod;
    }
    
    public int numberOfSequence(int n, int[] sick) {

        int holes = 0;
        long r = 1;
        holes += sick[0];
        
        for (int i = 0; i < sick.length - 1; i++) {
            int span = sick[i + 1] - sick[i] - 1;
            if (span > 0) {
                r = r * ksm(2, span - 1) % mod;
                holes += span;
                r = r * comb(holes, span) % mod;
            }
        }
        int m = sick.length;
        if (sick[m - 1] < n) {
            int span = (n - sick[m - 1] - 1);
            holes += span;
            r = r * comb(holes, span) % mod;
        }
        
        return (int)r;
        
    }
    
}
```

---

# 写在最后

