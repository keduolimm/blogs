
# 牛客小白月赛82 解题报告 | 珂学家 | 状压容斥 + 反悔堆 + 动态开点线段树

---
# 前言

![alt](https://uploadfiles.nowcoder.com/images/20231202/446702330_1701450189892/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价

这场小白真心难，E题成为这场的意难平，最后时候才理清。不过我是动态开点的线段树做法，时间刚好卡过。

C是状压+容斥，也可以用矩阵幂加速， D是反悔堆贪心。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 谜题：质数](https://ac.nowcoder.com/acm/contest/70845/A)

很有趣的一道题

两个质数(奇数）和一定是偶数，

所以需要特判质数2，可以配对7，使得2+7=9

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        if (n == 2) {
            System.out.println(7);
        } else if (n == 3){
            System.out.println(5);
        } else {
            System.out.println(3);
        }
    }

}
```

---

## [B. Kevin逛超市 2 (简单版本)](https://ac.nowcoder.com/acm/contest/70845/B)

分类讨论，
- n=1

  取A，B优惠券的优惠最大值。

- n>=2

  选取最大和次大两数，进行A(1) + B(2), B(1) + A(2)对比

```java
import java.io.*;
import java.util.*;
import java.util.function.Function;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int t = sc.nextInt();
        while (t-- > 0) {

            int n = sc.nextInt(), x = sc.nextInt(), y = sc.nextInt();
            Long[] arr = new Long[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextLong();
            }
            Arrays.sort(arr, Comparator.comparingLong(v -> -v));

            Function<Long, Long> A = (v) -> v * x;
            Function<Long, Long> B = (v) -> Math.max(0l, v - y) * 100;

            if (n == 1) {
                System.out.printf("%.6f\n", Math.min(A.apply(arr[0]), B.apply(arr[0])) / 100.0);
            } else {
                long f1 = A.apply(arr[0]), f2 = B.apply(arr[1]);
                long f3 = B.apply(arr[0]), f4 = A.apply(arr[1]);

                long res = Math.min(f1 + f2, f3 + f4);
                for (int i = 2; i < n; i++) {
                    res += arr[i] * 100;
                }
                System.out.printf("%.6f\n", res / 100.0);
            }
        }
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
    }

}
```

---

## [C. 被遗忘的书籍](https://ac.nowcoder.com/acm/contest/70845/C)

状压DP，这题可以用容斥来构建

即先求 ***不包含‘txt’的字符串方案数***

- 状态0， any，除1,2状态外的合法状态
- 状态1，以字母t结尾
- 状态2，以tx结尾

转移方程见代码

```java
import java.io.*;
import java.util.*;

public class Main {

    static long ksm(long b, long v, long mod) {
        long r = 1l;
        b = b % mod;
        while (v > 0) {
            if (v % 2== 1) r = r * b % mod;
            v /= 2;
            b = b * b %mod;
        }
        return r;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        // 预处理，提前计算1~20000的结果
        int n = 2 * 10_0001;
        long mod = 998244353;

        // any, t, tx,
        long[][] dp = new long[n][3];
        dp[0][0] = 25;
        dp[0][1] = 1;
        for (int i = 1; i < n; i++) {
            dp[i][0] = (dp[i - 1][0] * 25 % mod  + dp[i - 1][1] * 24 % mod + dp[i - 1][2] * 25 % mod) % mod;
            dp[i][1] = (dp[i - 1][0] + dp[i - 1][1]) % mod;
            dp[i][2] = dp[i - 1][1];
        }

        int t = sc.nextInt();
        while (t-- > 0) {
            int m = sc.nextInt();
            long ne = (dp[m - 1][0] + dp[m - 1][1] + dp[m - 1][2]) % mod;
            // 容斥，总方案数26^n - 不存在txt的方案数
            long res = ksm(26, m, mod) - ne;
            res = (res % mod + mod) % mod;
            System.out.println(res);
        }

    }

}
```

如果要继续优化的话，可以用矩阵幂。

---

## [D. Kevin逛超市 2 (困难版本)](https://ac.nowcoder.com/acm/contest/70845/D)

双变量组合求最优解，一般可以采用反悔堆来求解。

可以假定优先使用a类优惠券，然后b类优惠券去置换，这样能保证最优解一定在置换态的出现。

那如何构建反悔堆呢？

先来明确这题的核心：**交换价值**

也就说反悔的原动力，就是存在交换价值

$f(i) = max(0, arr[i] - y) - a[i] * x$, 用于构建已经置换为优惠券A对应的反悔堆。

那对于一张优惠券B，作用于j， 此时

A(i) + B(j) > B(i) + A(j) => B(i) - A(i) < B(j) - A(i) => f(i) < f(j)

当然这里情况有些复杂，还存在替代非交换的情况。

比如优惠券很多，但商品数太少。

这边直接比较 A(i)和B(i)即可。

当然这边有个技巧对应浮点数，就是整体乘以100， 最后结果在除以100，避免在浮点数上吃亏。

---

这题还有个想法，是否可以模拟退火来求解？

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();
        int t = sc.nextInt();
        while (t-- > 0) {
            int n = sc.nextInt();
            int a = sc.nextInt(), b = sc.nextInt();
            int x = sc.nextInt(), y = sc.nextInt();

            Long[] arr = new Long[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextLong();
            }
            Arrays.sort(arr, Comparator.comparingLong(v->-v));

            long res = 0;
            PriorityQueue<Long> pp = new PriorityQueue<>(Comparator.comparing(v -> v));
            int ma = Math.min(a, n);
            for (int i = 0; i < ma; i++) {
                pp.offer(Math.max(0, arr[i] - y) * 100l - arr[i] * x);
                res += arr[i] * x;
            }
            for (int i = ma; i < n; i++) {
                if (i - ma + 1 <= b) {
                    res += Math.max(0, arr[i] - y) * 100;
                } else {
                    res += arr[i] * 100;
                }
            }

            // 进行置换
            for (int i = 0; i < b && !pp.isEmpty(); i++) {
                if (ma < n) {
                    // 优惠券交换
                    long d1 = pp.peek();
                    long d2 = Math.max(0, arr[ma] - y) * 100l - arr[ma] * x;
                    if (d1 < d2) {
                        res += d1 - d2;
                        pp.poll();
                        pp.offer(d2);
                    }
                    ma++;
                } else {
                    // 优惠券取代，非置换，A类优惠券不被使用了
                    long d1 = pp.peek();
                    if (d1 < 0) {
                        res += d1;
                        pp.poll();
                    }
                }
            }

            System.out.printf("%.6f\n", res / 100.0);
        }
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
    }

}
```




---

## [E. Kevin的宝石](https://ac.nowcoder.com/acm/contest/70845/E)

赛中先用$O(n^2 * k)$的解法，测了下大致思路的正确性，得到一个大大的TLE，看来方向没错，而且有强数据。

- 模型转换

魔法宝石之间的匹配，存在并列结构，也存在嵌套结构。

由于嵌套结构的特殊性(没有没被消去的魔法宝石)，可以把嵌套结构看成一个整体。

这样在一个序列中，就是寻找几个互不重叠的子区间，其代价不超过K，但是总收益最大？

---

引入一个状态，$opt[i][s]$表示前i项中，最多消耗s金币的最大收益。

那这个状态转移怎么写呢？

![alt](https://uploadfiles.nowcoder.com/images/20231202/446702330_1701447161137/D2B5CA33BD970F64A6301FA75AE2EB22)


现在的问题是时间复杂度太高了，所以需要优化？

- 单调队列
- 数据结构优化

到底是那个呢？

再对上述的公式进行简化

![alt](https://uploadfiles.nowcoder.com/images/20231202/446702330_1701448599491/D2B5CA33BD970F64A6301FA75AE2EB22)

核心问题就是，如何简化那个Max，就算不能降到$O(1)$, 至少也能$O(log m)$

这就需要支持RMQ的结构来优化，且基于$T-C(i)$

不过由于$T-C(i)$可能是负数，且值域范围很大。

所以这边引入动态开点的线段树来作为RMQ。

- 这边还有一个难点，就是宝石是成双成对的

因此每个区间其魔法宝石必须是偶数，不能为奇数。

这边的解法是引入两个线段树，分别代表奇数位，偶数位有效。

---

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int t = sc.nextInt();
        while (t-- > 0) {
            int n = sc.nextInt(), k = sc.nextInt();

            long[] arr = new long[n];
            long[] brr = new long[n];
            List<Integer> tx = new ArrayList<>();
            for (int i = 0 ;i < n;i ++) {
                int tp = sc.nextInt();
                int v = sc.nextInt();
                if (tp == 1) {
                    arr[i] = v;
                } else {
                    // 魔法
                    brr[i] = v;
                    tx.add(i);
                }
            }

            long[] pre = new long[n + 1];
            long[] pre2 = new long[n + 1];
            for (int i = 0; i < n; i++) {
                pre[i + 1] = pre[i] + arr[i];
                pre2[i + 1] = pre2[i] + brr[i];
            }

            int m = tx.size();
            long[][] dp = new long[m + 1][k + 1];

            // 确定值域的范围
            long f = pre2[n] + k;
            // 引入两个线段树，用于奇偶划分
            DynamicSegTree root1 = new DynamicSegTree(-f, f);
            DynamicSegTree root2 = new DynamicSegTree(-f, f);

            for (int i = 0; i < m; i++) {
                int se = tx.get(i);
                for (int j = 0; j <= k; j++) {
                    // 不参与，则从上个状态直接转移过来
                    dp[i + 1][j] = Math.max(dp[i + 1][j], dp[i][j]);
                    if (j > brr[se]) {
                        // 如果参与
                        long pos = j - pre2[se + 1];
                        if (i % 2 == 0) {
                            long tv = root1.query(-f, pos);
                            dp[i + 1][j] = Math.max(dp[i + 1][j], tv + pre[se + 1]);
                        } else {
                            long tv = root2.query(-f, pos);
                            dp[i + 1][j] = Math.max(dp[i + 1][j], tv + pre[se + 1]);
                        }
                    }
                }
                if (i % 2 == 0) {
                    for (int j = 0; j <= k; j++) {
                        root2.updateSegTree(j - pre2[se], dp[i][j] - pre[se]);
                    }
                } else {
                    for (int j = 0; j <= k; j++) {
                        root1.updateSegTree(j - pre2[se], dp[i][j] - pre[se]);
                    }
                }
            }

            System.out.println(Arrays.stream(dp[m]).max().getAsLong());

        }

    }

    static
    class DynamicSegTree {
        static long DEFAULT_VALUE = Long.MIN_VALUE / 10;

        long l, r;
        DynamicSegTree left, right;

        long val;
        public DynamicSegTree(long l, long r) {
            this.l = l;
            this.r = r;
            this.val = DEFAULT_VALUE;
        }

        void updateSegTree(long p, long val) {
            if (isLeaf()) {
                this.val = Math.max(this.val, val);
                return;
            }
            long m = l + (r - l) / 2;
            if (p <= m) {
                if (this.left == null) {
                    this.left = new DynamicSegTree(l, m);
                }
                this.left.updateSegTree(p, val);
                this.val = Math.max(this.val, this.left.val);
            } else {
                if (this.right == null) {
                    this.right = new DynamicSegTree(m + 1, r);
                }
                this.right.updateSegTree(p, val);
                this.val = Math.max(this.val, this.right.val);
            }
        }

        long query(long ql, long qr) {
            if (isLeaf()) {
                return this.val;
            }
            if (ql <= l && qr >= r) {
                return this.val;
            }

            long m = l + (r - l) / 2;
            if (qr <= m) {
                if (this.left == null) {
                    return DEFAULT_VALUE;
                }
                return this.left.query(ql, qr);
            } else if (ql > m) {
                if (this.right == null) {
                    return DEFAULT_VALUE;
                }
                return this.right.query(ql, qr);
            } else {
                long lv = DEFAULT_VALUE;
                if (this.left != null) {
                    lv = this.left.query(ql, m);
                }
                long rv = DEFAULT_VALUE;
                if (this.right != null) {
                    rv = this.right.query(m + 1, qr);
                }
                return Math.max(lv, rv);
            }
        }
        boolean isLeaf() {
            return l == r;
        }

    }

}
```

时间复杂度为$O(n^2 log m)$, m为值域[-1e13, 1e13]

---




---


# 写在最后
![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231202/446702330_1701450231700/D2B5CA33BD970F64A6301FA75AE2EB22)




