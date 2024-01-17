
# 介绍

StringHash

如果遇到hash碰撞，则需要双hash

```java
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

```

# 题单

[3008. 找出数组中的美丽下标 II](https://leetcode.cn/problems/find-beautiful-indices-in-the-given-array-ii/description/)

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
