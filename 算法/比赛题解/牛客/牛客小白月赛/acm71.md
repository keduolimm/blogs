# 牛客小白月赛71 解题报告 | 珂学家 | 二维偏序 & 数论为王

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20230617/446702330_1686997952508/D2B5CA33BD970F64A6301FA75AE2EB22)

在那一天到来为止，我要用我的方法把优纪心的姿态传递下去。如果有一天有了孩子，一定会反复说给他听。在现实和幻想世界的狭缝剑，奇迹般耀眼辉煌的一位小小的女孩子的故事。

---

# 整体评价

数学场吧，E需要对gcd推导过程熟悉，同时兼具同余，质因子分解等知识点，F题是牛客经典的期望题，公式推导有些难。当然本场C题也挺有趣的，可以好好聊聊。

---

## [A. 猫猫与广告](https://ac.nowcoder.com/acm/contest/54877/A)

就是给你两个长方形A，B，B是否能覆盖A，允许翻转横竖方向。

可以先归一化，保证每个长方形的高不小于宽, 即$(H\geq W)$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int a = sc.nextInt(), b = sc.nextInt();
        int c = sc.nextInt(), d = sc.nextInt();
        int h1 = Math.max(a, b), w1 = Math.min(a, b);
        int h2 = Math.max(c, d), w2 = Math.min(c, d);
        System.out.println(h2 >= h1 && w2 >= w1 ? "YES" : "NO");
    }

}
```

---

## [B. 猫猫与密信](https://ac.nowcoder.com/acm/contest/54877/B)

就是一个字符串S，中间少了一个字符变成S‘，判断原先的S串中，是否有可能存在“love”这个单词.

其实本质上就是，枚举["love", "ove", "lve", "loe", "lov"]是否在S’中

因为词典数小，且单词长度也小，暴力匹配即可。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        String s = sc.next();
        boolean found = false;
        for (String p: new String[] {"love", "ove", "lve", "loe", "lov"}) {
            if (s.indexOf(p) != -1) {
                found = true;
                break;
            }
        }
        System.out.println(found ? "YES" : "NO");
    }

}

```

如果字典数大，且长度也长，那最好还是构建AC自动机来匹配求解。

---
## [C. 猫猫与数列](https://ac.nowcoder.com/acm/contest/54877/C)

很奇怪的数列，增长非常地迅猛

${a_n=a_{n-2}^{a_{n-1}}}$

求最大的n，使得 $a_n\leq 10^{18}$ 成立

这题难就难在long类型会溢出

所以这题，采用大数BigInteger来求解，因为迭代次数有限，所以能忍受这个。

同时指数的话，借助快速幂来实现

但是实际操作过程中，发现快速幂也超时了，很伤，尤其是第一个case(2 2)（出题者真心是良心）

```java
public static BigInteger ksm(long b, long v) {
    BigInteger r = BigInteger.valueOf(1);
    BigInteger cb = BigInteger.valueOf(b);
    while (v > 0) {
        if (v % 2 == 1) {
            r = r.multiply(cb);
        }
        cb = cb.multiply(cb);
        v /= 2;
    }
    return r;
}
```
因为cb这个变量增长的太迅猛了，使得每次操作代价非常的大，同时v本身也大，小小的几十次，就会导致cb指数级暴炸。

那怎么办呢？

得引入工程界的那一套，就是

> Fast Fail

就是但凡有个因子在乘积过程中，超过$10^18$，就提前退出

就因为这个改造，最终顺利过关，但真心不容易

```java
import java.io.BufferedInputStream;
import java.math.BigInteger;
import java.util.Scanner;

public class Main {

    static final BigInteger UP_LIMIT = BigInteger.valueOf((long)1e18);

    public static BigInteger ksm(long b, long v) {
        BigInteger r = BigInteger.valueOf(1);
        BigInteger cb = BigInteger.valueOf(b);
        while (v > 0) {
            if (v % 2 == 1) {
                r = r.multiply(cb);
                // 引入fast fail机制
                if (r.compareTo(UP_LIMIT) > 0) return null;
            }
            cb = cb.multiply(cb);
            v /= 2;

            if (v > 0) {
                // 引入fast fail机制
                if (cb.compareTo(UP_LIMIT) > 0) return null;
            }

        }
        return r;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        long p = sc.nextLong(), q = sc.nextLong();
        BigInteger prev = BigInteger.valueOf(p);
        BigInteger next = BigInteger.valueOf(q);

        int i = 2;
        while (true) {
            BigInteger next3 = ksm(prev.longValue(), next.longValue());
            if (next3 == null || next3.compareTo(UP_LIMIT) > 0) break;
            prev = next;
            next = next3;
            i++;
        }
        System.out.println(i);
    }

}

```


---
## [D. 猫猫与主人](https://ac.nowcoder.com/acm/contest/54877/D)

二维偏序的题，不过相对比较简单

就是每个猫有个属性(a, c), 主人有属性(b, d)

需要保证 b>=c && a>=d 情况下，每个猫可以找到最好主人的b值

其实就是以猫为主视角，对主人按属性d排序$[(b_{k1}, d_{k1}), ..., (b_{kn}, d_{kn})]$

然后找到$ki$(a>=d)的前缀区间$[0, ki]$，求b的最大值，就是区间最值问题

当然因为这边是前缀区间(从开头起），所以这边是可以dp数组维护前缀最大值。

也可以用树状数组维护，就是有点浪费。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.Map;
import java.util.StringTokenizer;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class Main {

    // 非差分型
    static class BIT {
        int n;
        int[] arr;
        public BIT(int n) {
            this.n = n;
            this.arr = new int[n + 1];
        }

        public void update(int p, int d) {
            while (p <= n) {
                arr[p] = Math.max(arr[p], d);
                p += lowbit(p);
            }
        }

        // 前缀查询
        public int query(int p) {
            int res = 0;
            while (p > 0) {
                res = Math.max(arr[p], res);
                p -= lowbit(p);
            }
            return res;
        }

        int lowbit(int x) {
            return x & -x;
        }

    }

    public static void main(String[] args) {
        AReader sc = new AReader();
        int n = sc.nextInt();
        int m = sc.nextInt();

        int[][] cat = new int[n][2];
        for (int i = 0; i < n; i++) {
            cat[i][0] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            cat[i][1] = sc.nextInt();
        }

        int[][] user = new int[m][2];
        for (int i = 0; i < m; i++) {
            user[i][0] = sc.nextInt();
        }
        for (int i = 0; i < m; i++) {
            user[i][1] = sc.nextInt();
        }

        Arrays.sort(user, (a, b) -> {
            if (a[1] != b[1]) return a[1] < b[1] ? -1 : 1;
            return 0;
        });

        BIT bit = new BIT(user.length);
        TreeMap<Integer, Integer> ts = new TreeMap<>();
        for (int i = 0; i < user.length; i++) {
            ts.merge(user[i][1], i + 1, Math::max);
            bit.update(i + 1, user[i][0]);
        }

        int[] res = new int[cat.length];
        for (int i = 0; i < cat.length; i++) {
            int a = cat[i][0], b = cat[i][1];
            Map.Entry<Integer, Integer> e = ts.floorEntry(a);

            if (e == null) {
                res[i] = -1;
            } else {
                int mz = bit.query(e.getValue());
                if (mz >= b) {
                    res[i] = mz;
                } else {
                    res[i] = -1;
                }
            }
        }

        String line = Arrays.stream(res).mapToObj(String::valueOf).collect(Collectors.joining(" "));
        System.out.println(line);

    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }

}

```


---
## [E. 猫猫与数学](https://ac.nowcoder.com/acm/contest/54877/E)


求${gcd(a+c, b+c) \neq 1}$，最小的非负数c?

题意很简洁，不过说真的，真的很难下手

还是从gcd的推演变形着手，就$gcd(b, a) = gcd(b, a-b)$

因此 ${gcd(b+c, a+c) => gcd(b+c, a-b)}$

由于 $gcd(b+c, a-b)\neq 1$ 所以 b+c 一定可以整除a-b的某个质因子

这题的思路，就是从这里进行切入的。

枚举a-b的质因子p

$c = ((a - 2 * b) \% p + p) \% p$

取最小的c就行

而质因子拆解，其时间复杂度为$O(\sqrt{(a - b)}$

```java
import java.io.BufferedInputStream;
import java.math.BigInteger;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        long a = sc.nextLong();
        long b = sc.nextLong();

        if (a < b) {
            long t = a;
            a = b;
            b = t;
        }

        long d = a - b;
        if (d == 0) {
            System.out.println(a == 1 ? 1 : 0);
        } else if (d == 1) {
            System.out.println(-1);
        } else {
            long ans = Long.MAX_VALUE;
            long dt = d;
            for (long i = 2; i * i <= d; i++) {
                if (d % i == 0) {
                    // b + c,  a - b
                    // b + c = d % i
                    // c = ((d % i - b % i) % i + i) % i
                    ans = Math.min(ans, ((dt-b) % i + i) % i);
                    while (d % i == 0) d/=i;
                }
            }
            if (d > 1) {
               ans = Math.min(ans, ((dt-b) % d + d) % d);
            }
            System.out.println(ans);
        }

    }

}
```

---
## [F. 猫猫与宝石](https://ac.nowcoder.com/acm/contest/54877/F)

这题还是参考了出题人的解答

就是引入参与度

那最终的${E=\sum_{i=1} ^{i=n}1/2 a_i * ((n-1)/2 + 1)}$

再简化后变为 $E=(n+1)/4 * \sum_{i=1} ^{i=n} a_i$

```java
import java.io.BufferedInputStream;
import java.math.BigInteger;
import java.util.Scanner;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Main {

    public static void main(String[] args) {
      
        long mod = 998244353l;
      
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        long[] pre = new long[n + 1];
        for (int i = 0; i < n; i++) {
            pre[i + 1] = pre[i] + arr[i];
        }

        long inv4 = BigInteger.valueOf(4).modInverse(BigInteger.valueOf(mod)).longValue();
      
        String line = IntStream.range(0, n).mapToObj(x -> {
            long r = pre[x + 1] % mod * (x + 2) % mod * inv4 % mod;
            return String.valueOf(r);
        }).collect(Collectors.joining(" "));
      
        System.out.println(line);
      
    }


}
```

---

# 写在最后

初次见面，我是结城明日奈。

![alt](https://uploadfiles.nowcoder.com/images/20230617/446702330_1686997978173/D2B5CA33BD970F64A6301FA75AE2EB22)


