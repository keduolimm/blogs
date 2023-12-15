# 牛客小白月赛77 解题报告 | 珂学家 | 反悔堆 + 字符串hash + 二进制递推

---

# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230904/446702330_1693842939308/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价

虽然E题测试数据有误，导致比赛期间非c++语言输入异常，但是瑕不掩瑜。题目质量还是蛮高的，D题设计卡单Hash，还是很用心的，E题这个反悔堆出的也很精彩，F题的官解做法也让人眼前一亮。

---
## [A. 小Why的方阵](https://ac.nowcoder.com/acm/contest/64384/A)

由于对称性，可以假定改动的值在左上角

x + a[0][1] = a[1][0] + a[1][1]

x + a[1][0] = a[0][1] + a[1][1]

恒成立的条件为 a[0][1] == a[1][0]

因此猜测，只要存在一个对角线元素相等，则必然可以构造解

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int a00 = sc.nextInt(), a01 = sc.nextInt();
        int a10 = sc.nextInt(), a11 = sc.nextInt();
        if (a00 == a11 || a01 == a10) {
            System.out.println("YES");
        } else {
            System.out.println("NO");
        }
    }

}

```

---
## [B. 小Why的循环置换](https://ac.nowcoder.com/acm/contest/64384/B)

很经典的一个知识点，置换环，它由多个互相独立的环构成.

对于某个成员数k的环，其最多需要k-1次，才能调整为自然序

这样的话，m个环，即需要

$\sum_{i=0}^{i=m} (k_i - 1) = n - m$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int m = sc.nextInt();
        System.out.println(n - m);
    }

}
```

---
## [C. 小Why的商品归位](https://ac.nowcoder.com/acm/contest/64384/C)

这是一道非常有趣的题

如果数据量小的话，感觉可以模拟，按 目标库编号 小的优先放购物车。

这种一入一出，是不是特别像 差分的做法

因此我们可以猜测，轮次和差分的最大数有关

> $res = \max\ ceiling(\sum_{i=0}^{i=j} diff[j] / k)$

不过这边取出购物车在先，从柜台取出在后，因此右端点不需要+1， 这是唯一和差分有区别的地方

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int m = sc.nextInt();
        int k = sc.nextInt();

        int[] diff = new int[n + 2];
        for (int i = 0; i < m; i++) {
            int from = sc.nextInt(), to = sc.nextInt();
            diff[from]++;
            diff[to]--; // 不需要+1
        }

        int ans = 0;
        int acc = 0;
        for (int i = 1; i <= n; i++) {
            acc += diff[i];
            ans = Math.max(ans, (acc + k - 1) / k);
        }
        System.out.println(ans);
    }

}
```


---
## [D. 小Why的密码锁](https://ac.nowcoder.com/acm/contest/64384/D)

这题第一感觉就是字符串hash

按照固定长度m，构建一个Map<字符串hash值，个数>

这题难点在于
- 子串不能重叠
- 数量恰好为k个

因此单纯地保存个数，可能不够，需要保存对应的区间，Map<字符串hash值，list<int[]>>

那保存区间列表后，又该如何处理呢？

可以根据贪心，挑选最多的不重叠个区间，如果恰好为k，那就是可能的密码串

不过这题卡单hash，反正试了13,17都不行，还好有预案，就是试双hash

赛后有群友反馈，是针对mod=1000000007，998244353设计测试数据了

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt(), m = sc.nextInt(), k = sc.nextInt();
        char[] str = sc.next().toCharArray();

        Map<Long, List<int[]>> hash = new HashMap<>();

        long base = 1l;
        long acc = 0;
        int p = 13;


        long base1 = 1l;
        long acc1 = 0;
        int p1 = 1771;

        long mod = 10_0000_0007l;
        for (int i = 0; i < m - 1; i++) {
            acc = (acc * p % mod + (str[i] - '0')) % mod;
            base = base * p % mod;

            acc1 = (acc1 * p1 % mod + (str[i] - '0')) % mod;
            base1 = base1 * p1 % mod;
        }

        for (int i = m - 1; i < n; i++) {
            acc = (acc * p % mod + (str[i] - '0')) % mod;
            acc1 = (acc1 * p1 % mod + (str[i] - '0')) % mod;

            hash.computeIfAbsent((acc1 << 32)|acc, x -> new ArrayList<>())
              .add(new int[] {i - m + 1, i});

            acc = (acc - base * (str[i - (m - 1)] - '0') % mod) % mod;
            acc = (acc + mod) % mod;

            acc1 = (acc1 - base1 * (str[i - (m - 1)] - '0') % mod) % mod;
            acc1 = (acc1 + mod) % mod;
        }

        int ans = 0;
        for (Map.Entry<Long, List<int[]>> kv: hash.entrySet()) {
            List<int[]> list = kv.getValue();
            if (list.size() >= k) {
                int last = -1;
                int nk = 0;
                for (int[] v: list) {
                    if (v[0] > last) {
                        last = v[1];
                        nk++;
                    }
                }
                if (nk == k) {
                    ans++;
                }
            }
        }
        System.out.println(ans);

    }

}
```

--- 
## [E. 小Why的简单加减](https://ac.nowcoder.com/acm/contest/64384/E)

这题负数必须和前面的数配对处理，才能变成非负数，这是大前提。

同时有个最多k次操作的限制，求数组中最多的非负数个数

对于某个负数要变成非负数(0)，需要分类讨论

- 前排都已经变成非负数，

  |arr[i]| <= min(前缀和, k + 负数的前缀和)

- 前排至少有一个负数没有变非负数

  |arr[i]| <= k + 选中的负数前缀和

如果两者条件都不能满足了，则需要考虑，是否可以置换前排 已被非负数的 负数，看谁代价更小

这个置换过程，就是反悔过程，而维护替换的数据结构就是堆(最大堆）

因此这题，思路其实非常的明确，就是反悔贪心

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        long k = sc.nextLong();
        int nk = 0;
        long[] arr = new long[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
            if (arr[i] >= 0) nk++;
        }
        // 反悔堆
        PriorityQueue<Long> pp = new PriorityQueue<>(Comparator.comparing(x -> -x));

        long acc = 0;
        long pacc = 0;
        int minus = 0;

        for (int i = 0; i < n; i++) {
            if (arr[i] >= 0) {
                acc += arr[i];
            } else {
                minus++;
                long tv = -1l * arr[i];
                if ((pacc + tv <= k) && (pacc + tv <= acc || pp.size() + 1 < minus)) {
                    pp.offer(tv);
                    pacc += tv;
                } else {
                    if (!pp.isEmpty() && pp.peek() > tv) {
                        pacc -= pp.poll();
                        pp.offer(tv);
                        pacc += tv;
                    }
                }
            }
        }
        System.out.println(nk + pp.size());
    }

}
```

---
## [F. 小Why的数论测试](https://ac.nowcoder.com/acm/contest/64384/F)

对于这类从某一个二进制变成另一个二进制的问题

最好的方式还是递推

如果仅仅考虑
- 加1操作
- 乘2操作

那这个最小代价，其实可以立马求解出来。

关键是sqrt操作操作

可以递推从a到sqrt(b)，状态数最多10^6, 可控，同时大于sqrt(b)不存在平方操作，可以保证最优解。

期间可以+1，x2，平方迁移，然后评估其非平方的总代价

这样时间复杂度为 sqrt(b)

```java
import java.io.*;
import java.util.*;

public class Main {

    static long evaluate(long a, long b) {
        if (a > b) return Integer.MAX_VALUE;

        int n1 = 64 - Long.numberOfLeadingZeros(a);
        int n2 = 64 - Long.numberOfLeadingZeros(b);

        int dn = n2 - n1;
        long b1 = b >>> dn;

        if (a <= b1) {
            return (b1 - a) + dn + Long.bitCount(b - (b1 << dn));
        } else {
            return ((b1 << 1) - a) + (dn - 1) + Long.bitCount(b - (b1 << dn));
        }
    }

    public static void main(String[] args) {

        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int kase = sc.nextInt();
        while (kase-- > 0) {
            long a = sc.nextLong();
            long b = sc.nextLong();

            long ans = evaluate(a, b);

            if (a <= b / a) {
                int mz = (int) Math.sqrt(b);
                long[] vis = new long[mz + 1];
                Arrays.fill(vis, Integer.MAX_VALUE);

                int ia = (int) a;
                vis[ia] = 0;
                for (int i = ia; i <= mz; i++) {
                    ans = Math.min(ans, vis[i] + evaluate(i, b));

                    if (i + 1 <= mz) {
                        vis[i + 1] = Math.min(vis[i + 1], vis[i] + 1);
                    }
                    if (i * 2 <= mz) {
                        vis[i * 2] = Math.min(vis[i * 2], vis[i] + 1);
                    }

                    if ((long) i * i <= mz) {
                        vis[i * i] = Math.min(vis[i * i], vis[i] + 1);
                    } else {
                        // 这边需要保护
                        ans = Math.min(ans, vis[i] + 1 + evaluate((long) i * i, b));
                    }
                }
            }
            System.out.println(ans);
        }

    }

}
```

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20230904/446702330_1693842982715/D2B5CA33BD970F64A6301FA75AE2EB22)









