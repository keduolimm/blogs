
# 牛客小白月赛78 解题报告 | 珂学家 | 数学场 + 线性同余方程

---
# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230916/446702330_1694849543749/D2B5CA33BD970F64A6301FA75AE2EB22)

---

# 整体评价

数学场，牛客无论周赛，小白，练习赛，都喜欢出数论题。C题钻牛角尖了，二分+容斥一条路走到黑了，太惨了。D题是找规律题，找到了就容易了。E题是板子题，思路很容易想到，但是求解要么有板子，要么得有技巧。

---
## A.

---
## B.

---
## [C. 第K小表示数](https://ac.nowcoder.com/acm/contest/65157/C)

这种类型的题，有两种思路

- 多路归并
- 二分 + 容斥

前者适合k值较小，后者是通解，大小通吃。
不过这题，按容斥来解，感觉毫无头绪，T_T

所以只能回到多路归并的思路去解

借助优先队列，每次扩展+a/+b，去重模拟即可。

当然这题，如果a=b, 直接返回k*a即可

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int k = sc.nextInt();
        long a = sc.nextLong();
        long b = sc.nextLong();

        if (a == b) {
            System.out.println(k * a);
        } else {
            Set<Long> tx = new HashSet<>();
            PriorityQueue<Long> pp = new PriorityQueue<>(Comparator.comparing(x -> x));
            pp.offer(a); pp.offer(b);
            tx.add(a); tx.add(b);

            while (k > 1 && !pp.isEmpty()) {
                long t = pp.poll();
                if (tx.add(t + a)) {
                    pp.offer(t + a);
                }
                if (tx.add(t + b)) {
                    pp.offer(t + b);
                }
                k--;
            }
            System.out.println(pp.poll());
        }

    }

}
```

---

## D.

这个属于找规律

纯数学题



---

## [E. 自动计算机](https://ac.nowcoder.com/acm/contest/65157/E)

求次数，其实可以转化如下公式, S为整个数组的累加和，k为整个数组的个数

> x + k * S + S[i] = 0 % n   => k*S = (-x - S[i]) % n

求解得到最小的非负数整数k，然后k*m + (i+1)

这属于线性同余方程解

但是针对 模数的大小，有两类解法

如果模数较小，可以预处理，因为根据鹊巢原理，最多模数次

如果模数较大，只能借助同余方程来求解

感觉牛客对线性同余方程，普遍模数较小，因此预处理硬算即可。

### 枚举模拟

因为模数较小，最多计算模数次(鹊巢原理)，即可预处理出同余方程的解

```java
import java.io.*;
import java.util.*;

public class Main {

    static int wrap(long v, int n) {
        return (int)((v % n + n) % n);
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), m = sc.nextInt(), x = sc.nextInt();

        long[] pre = new long[m + 1];
        int[] arr = new int[m];
        for (int i = 0; i < m; i++) {
            arr[i] = sc.nextInt();
            pre[i + 1] = pre[i] + arr[i];
        }

        // 前后最分解
        long acc = wrap(pre[m], n);

        Map<Integer, Long> hash = new HashMap<>();
        for (int i = 0; i < n; i++) {
            int tacc = wrap(acc % n * i % n, n);
            if (hash.containsKey(tacc)) {
                // 循环节, 因为取模n, 所以最多n个
                break;
            }
            hash.put(tacc, (long)i * m);
        }

        long inf = Long.MAX_VALUE / 10;
        long ans = hash.getOrDefault(wrap(-x, n), inf);

        for (int i = 0; i < m; i++) {
            long t = -x - pre[i + 1];
            long r1 = hash.getOrDefault(wrap(t, n), inf);
            ans = Math.min(ans, r1 + (i + 1));
        }
        System.out.println(ans == inf ? -1: ans);

    }
}
```

### 扩展欧几里得

关于 扩展欧几里得 求解同余方程

https://oi-wiki.org/math/number-theory/linear-equation/

![alt](https://uploadfiles.nowcoder.com/images/20230916/446702330_1694848810934/D2B5CA33BD970F64A6301FA75AE2EB22)


```java
import java.io.*;
import java.util.*;
import java.util.function.Function;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), m = sc.nextInt();
        long x = sc.nextInt();

        long[] pre = new long[m + 1];
        int[] arr = new int[m];
        for (int i = 0; i < m; i++) {
            arr[i] = sc.nextInt();
            pre[i + 1] = pre[i] + arr[i];
        }

        Function<Long, Long> fp = t -> (t % n + n) % n;
        long acc = fp.apply(pre[m]);

        long inf = Long.MAX_VALUE / 10;
        long ans = inf;
        CongruentLinearEquation p1 = new CongruentLinearEquation();
        if (p1.solve(acc, n, fp.apply(-x))) {
            ans = p1.x * m;
        }

        for (int i = 0; i < m; i++) {
            long t = -x - pre[i + 1];
            CongruentLinearEquation p2 = new CongruentLinearEquation();
            if (p2.solve(acc, n, fp.apply(t))) {
                ans = Math.min(ans, (long)p2.x * m + (i + 1));
            }
        }
        System.out.println(ans == inf ? -1 : ans);
    }

    // 同余线性方程
    static
    class CongruentLinearEquation {
        long x, y, t;
        long exGcd(long a, long b) {
            if (b == 0) {
                x = 1; y = 0;
                return a;
            }
            long ret = exGcd(b, a % b);
            long x1 = y, y1 = x - a / b * y;
            x = x1; y = y1;
            return ret;
        }

        // ax = c % b
        // ax + by = c
        public boolean solve(long a, long b, long c) {
            // https://oi-wiki.org/math/number-theory/linear-equation/
            long d = exGcd(a, b);
            if(c % d != 0) return false;

            long k = c / d;
            x *= k; y *= k;

            // 同解为
            // x' = x + b/d * z,  z \in Z 整数
            // 最小的非负整数为
            // t = b/d
            // x' = (x % t + t) % t

            this.t = b / d;
            x = (x % t + t) % t;
            return true;
        }
    }

}
```


---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230916/446702330_1694849705772/D2B5CA33BD970F64A6301FA75AE2EB22)

