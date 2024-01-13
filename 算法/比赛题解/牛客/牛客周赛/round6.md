# 牛客周赛 Round 6 解题报告 | 珂学家 | 数学场

---
# 前言

![alt](https://uploadfiles.nowcoder.com/images/20230806/446702330_1691332353364/D2B5CA33BD970F64A6301FA75AE2EB22)

一切都是命运的安排。

---

# 整体评价

这场整体感觉有点简单，D题感觉不错，E题应该是超纲了。整场还是偏数学，个人还是喜欢Round 4/Round 5.

---

## [A. 游游的数字圈](https://ac.nowcoder.com/acm/contest/62622/A)

简单模拟题

- 0,6,9对应一个圆圈
- 8对应2个圆圈

```java []
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        int ans = 0;
        for (char c: str) {
            int p = c - '0';
            if (p == 0 || p == 6 || p == 9) ans++;
            else if (p == 8) ans += 2;
        }
        System.out.println(ans);
    }

}
```
```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    
    string s;
    cin >> s;
    int res = 0;
    for (char c: s) {
        if (c == '0') res += 1;
        else if (c=='6') res += 1;
        else if (c == '8') res+= 2;
        else if (c == '9') res+= 1;
    }
    cout << res << endl;
    return 0;
}
```


---

## [B. 竖式乘法](https://ac.nowcoder.com/acm/contest/62622/B)

阅读题吧，

就是b的每个位数乘以a的累计和

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int kase = sc.nextInt();
        while (kase-- > 0) {
            long a = sc.nextInt();
            long b = sc.nextInt();

            long ans = 0;
            while (b > 0) {
                long t = b % 10;
                ans += a * t;
                b /= 10;
            }
            System.out.println(ans);
        }
    }

}
```

---

## [C. 游游的数值距离](https://ac.nowcoder.com/acm/contest/62622/C)

这题还是稍有点头痛的

化简公式 $|x!\times y - y - n|$ 等价于 $|(x! - 1) * y - n|$

求这个绝对值最小化时的，x,y值

由于n固定，那如何求解x,y呢？

可以发现x!膨胀非常的快，所以可以从x枚举入手

然后反解y值，因为是绝对值

所以 $y\in \{ n/(x! - 1), (n+x!-2)/(x! - 1) \}$

当然这题还有额外的限制，就是x,y都不能等于2，且为正整数

关键还是找对切入点

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        long n = sc.nextInt();
        long diff = Long.MAX_VALUE;
        long tx = 0, ty = 0;

        // 枚举x就行
        long x = 1l;
        for (int i = 1; x - 1 <= n; i++) {
            x = x * i;

            if (i == 1) {
                // 1 要 特判
                if (diff > n) {
                    diff = n;
                    tx = 1; ty = 1;
                }
            }

            if (x > 1 && i != 2) {
                long y1 = n / (x - 1);
                long y2 = (n + x - 2) / (x - 1);

                if (y1 != 2 && y1 > 0) {
                    long t = Math.abs((x - 1) * y1 - n);
                    if (diff > t) {
                        diff = t;
                        tx = i;
                        ty = y1;
                    }
                }
                if (y2 != 2 && y2 > 0) {
                    long t2 = Math.abs((x - 1) * y2 - n);
                    if (diff > t2) {
                        diff = t2;
                        tx = i;
                        ty = y2;
                    }
                }
            }
        }
        System.out.println(tx + " " + ty);
    }
    
}
```
---

## [D. 游游的k-好数组](https://ac.nowcoder.com/acm/contest/62622/D)

这题其实有点意思，但是如果找对正确的切入点，那这题还是蛮简单的。

因为最终所有的k长度的区间，其区间和相等.

比如 $[a_1, a_2, ..., a_k], [a_2,a_3,...a_k,a_{k+1}]$

可以推导出$a_1 == a_{k+1}$

从中我们可以推导出按取模k，进行分组

每组中元素必须相等

可以先按必须的给予操作（从x中获得）

如果最终的操作数 大于 x，则无解

如果小于等于, 则剩余x，可以贪心给予某一组，从而求解最大值。

```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt(), k = sc.nextInt(), x = sc.nextInt();
            int[] arr = new int[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextInt();
            }

            // 进行分组
            long[] last = new long[k];
            List<Integer>[]g = new List[k];
            Arrays.setAll(g, t -> new ArrayList<>());
            for (int i = 0; i < n; i++) {
                g[i % k].add(arr[i]);
            }

            for (int i = 0; i < k; i++) {
                long mv = g[i].get(0);
                for (long v: g[i]) {
                    mv = Math.max(mv, v);
                }
                last[i] = mv;
                for (int j = 0; j < g[i].size(); j++) {
                    x -= (mv - g[i].get(j));
                }
            }

            if (x < 0) {
                System.out.println(-1);
            } else {
                long ans = 0;
                for (int i = 0; i < k; i++) {
                    ans = Math.max(ans, last[i] + x / g[i].size());
                }
                System.out.println(ans);
            }
        }

    }

}
```



---

## [E. 小红的循环节长度](https://ac.nowcoder.com/acm/contest/62622/E)

这题应该超纲了

我大概知道的知识点是, 如果除数只有2,5的因子，则一定是有限个数的有理数，否则存在循环节。

[这是知乎上的一个讨论](https://www.zhihu.com/question/462266812)

大概意思为，混合小数中，不循环部分和2,5因子有关，循环节则和q的欧拉函数因子有关

> 特别要注意，求快速幂时，因为模数超过int范围，则两个long相乘会直接溢出，因为需要int128，当然java需用大数类。

```java
import java.io.BufferedInputStream;
import java.math.BigInteger;
import java.util.Scanner;

public class Main {

    static long gcd(long a, long b) {
        return b == 0 ? a : gcd(b, a % b);
    }

    static long phi(long v) {
        long r = v;
        for (long i = 2; i <= v/i; i++) {
            if (v % i == 0) {
                r = r / i * (i - 1);
                while (v % i == 0) v /= i;
            }
        }
        if (v > 1) r = r / v * (v - 1);
        return r;
    }

    // 这边需要特别注意，要用大数，因为q这个模数超过int, long * long 会溢出
    static long ksm(long b, long v, long q) {
        BigInteger bigQ = BigInteger.valueOf(q);
        BigInteger r = BigInteger.valueOf(1l);
        BigInteger base = BigInteger.valueOf(b).mod(bigQ);
        while (v > 0) {
            if (v % 2 == 1) {
               r = r.multiply(base).mod(bigQ);
            }
            v /= 2;
            base = base.multiply(base).mod(bigQ);
        }
        return r.mod(bigQ).longValue();
    }

    public static void main(String[] args) {

        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        long p = sc.nextLong(), q = sc.nextLong();

        long gv = gcd(p, q);
        q /= gv;

        int cnt2 = 0, cnt5 = 0;
        while (q % 2 == 0) { q /= 2; cnt2++; }
        while (q % 5 == 0) { q /= 5; cnt5++; }

        if (q == 1) {
            System.out.println(-1);
        } else {
            long ans = Long.MAX_VALUE;
            long nq = phi(q);
            for (long i = 1; i <= nq / i; i++) {
                if (nq % i == 0) {
                    if (ksm(10, i, q) == 1) ans = Math.min(ans, i);
                    if (ksm(10, nq / i, q) == 1) ans = Math.min(ans, nq / i);
                }
            }
            System.out.println("" + Math.max(cnt2, cnt5) + " " + ans);
        }
    }

}

```

---

# 写在最后

不辅助就只能回去继承家业了。

![alt](https://uploadfiles.nowcoder.com/images/20230806/446702330_1691332398665/D2B5CA33BD970F64A6301FA75AE2EB22)



