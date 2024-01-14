

# 第 380 场周赛 解题报告 | 珂学家 | 数位DP & 二分 + 字符串Hash

---

# 前言

![image.png](https://pic.leetcode.cn/1705205234-jCpYmA-image.png)


---

# 整体评价

感觉T3更难些，T4太直接了，一般的KMP/StringHash基本就够用了。

上周T4出数位DP，估计是为T3打了一个铺垫。


---

## [A. 最大频率元素计数](https://leetcode.cn/contest/weekly-contest-380/problems/count-elements-with-maximum-frequency/)

思路: 模拟即可

```java []
class Solution {
    public int maxFrequencyElements(int[] nums) {
        Map<Integer, Integer> hash = new HashMap<>();
        for (int v: nums) {
            hash.merge(v, 1, Integer::sum);
        }
        int fre = hash.values().stream().mapToInt(Integer::valueOf).max().getAsInt();
        return (int)hash.values().stream().filter(x -> x == fre).count() * fre;
    }
}
```


---
## [B. 找出数组中的美丽下标 I](https://leetcode.cn/contest/weekly-contest-380/problems/find-beautiful-indices-in-the-given-array-i/)

和T4一起讲

---
## [C. 价值和小于等于 K 的最大数字](https://leetcode.cn/contest/weekly-contest-380/problems/maximum-number-that-sum-of-the-prices-is-less-than-or-equal-to-k/)

思路: 数位DP+二分

这题题目真的绕，有点头痛

其二分阶段性还是非常的明显的，就是如何写check的问题。

数位DP，可以递推着写，就是沿着边界计算，因为这是二进制，反而比 传统的数位DP（记忆化搜索）更简洁。

其核心为

$$定义左侧1的有效位数acc1，右侧的长度left$$

$$s1 = 前置1的个数  * 右侧可构造的个数 = acc1 * 2 ^ {left}$$

$$s2 = 右侧可构造的个数中包含1的总个数 = 2 ^ {left} * (left/x) / 2$$

```java []
class Solution {

    boolean check(long m, long k, int x) {
        String s = new StringBuilder(Long.toString(m, 2)).reverse().toString();
        char[] str = s.toCharArray();

        long res = 0;
        int acc = 0;
        for (int i = str.length - 1; i >= 0; i--) {
            int p = str[i] - '0';
            if (p == 1) {
                int left = i;
                res += (1l << left) * acc;
                res += (1l << left) * (left / x) / 2;
            }
            if ((i + 1) % x == 0 && p == 1) acc++;
        }
        res += acc;

        return res <= k;
    }

    // 二分+数位DP
    public long findMaximumNumber(long k, int x) {
        long l = 0, r = 1l << 50;
        while (l <= r) {
            long m = l + (r - l) / 2;
            if (check(m, k, x)) {
                l = m + 1;
            } else {
                r = m - 1;
            }
        }

        return r;
    }

}
```

如果非要用记忆化的方式，感觉状态应该是

$$(第i位，前缀1的有效个数)$$


---
## [D. 找出数组中的美丽下标 II](https://leetcode.cn/contest/weekly-contest-380/problems/find-beautiful-indices-in-the-given-array-ii/)

思路： KMP/StringHash

因为手头有StringHash板子，所以用了StringHash求解

当然用了双Hash, 避免被Hack.


```java []
class Solution {
    public List<Integer> beautifulIndices(String s, String a, String b, int k) {
        long mod = (long)1e9 + 7;
        int p1 = 13, p2 = 17;
        StringHash h1 = new StringHash(s, p1, mod);
        StringHash h2 = new StringHash(s, p2, mod);

        List<Integer> lists = new ArrayList<>();
        long t1 = StringHash.evaluate(a, p1, mod);
        long t2 = StringHash.evaluate(a, p2, mod);
        for (int i = 0; i <= s.length() - a.length(); i++) {
            if (h1.query(i, i + a.length() - 1) == t1 && h2.query(i, i + a.length() - 1) == t2) {
                lists.add(i);
            }
        }

        TreeSet<Integer> ts = new TreeSet<>();
        long t3 = StringHash.evaluate(b, p1, mod);
        long t4 = StringHash.evaluate(b, p2, mod);
        for (int i = 0; i <= s.length() - b.length(); i++) {
            if (h1.query(i, i + b.length() - 1) == t3
                    && h2.query(i, i + b.length() - 1) == t4) {
                ts.add(i);
            }
        }

        List<Integer> res = new ArrayList<>();
        for (int i: lists) {
            if (ts.contains(i)) {
                res.add(i);
            } else {
                Integer k1 = ts.floor(i);
                Integer k2 = ts.ceiling(i);
                if (k1 != null && Math.abs(k1 - i) <= k) res.add(i);
                else if (k2 != null && Math.abs(k2 - i) <= k) res.add(i);
            }

        }
        return res;
    }

    static
    class StringHash {
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
}
```


---

# 写在最后


![image.png](https://pic.leetcode.cn/1705205212-bNkSVg-image.png)
