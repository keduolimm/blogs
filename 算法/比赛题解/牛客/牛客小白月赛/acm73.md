# 牛客小白月赛73 解题报告 | 珂学家 | 三指针 + DP构造 + 数学期望

---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20230615/446702330_1686820856422/D2B5CA33BD970F64A6301FA75AE2EB22)

比起一直躲在初始之街，慢慢腐朽，还不如到最后一刻都保持自身的存在。即便是死在怪兽手上，我也不想对这个游戏，这个世界认输，无论如何也不会！

---

# 整体评价

前面几题很简单，连思维难度都没，后面几题有点意思，D/E是脑筋急转弯，可以用三指针快速求解，F是经典的DP构造解（需要降维），G是数学期望题。

---
## [A. 最小的数字](https://ac.nowcoder.com/acm/contest/57683/A)

找到一个数${x\ge n}$, 且x是3的倍数

顺序枚举n, n+1, n+2这三个数好了，最早的一个必是解

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        for (int i = 0; i < 3; i++) {
            if ((n + i) % 3 == 0) {
                System.out.println((n + i));
                break;
            }
        }
    }

}
```
---
## [B. 优美的GCD](https://ac.nowcoder.com/acm/contest/57683/B)

求两个不同的数x,y, 其$gcd(x,y)=n$

根据题意可知，${x=k_1*n, y=k_2*n, 且gcd(k_1,k_2)=1}$

两个相邻的数，天然互质，所以取 $k_1=1, k_2=2$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            // x=1*n, y=2*n
            System.out.println((n) + " " + (n * 2));
        }
    }

}
```

---
## [C. 优美的序列](https://ac.nowcoder.com/acm/contest/57683/C)

能否重排一个序列$a$

使得$|a[i]-a[j]|\geq |i - j|$恒成立

对于这样的等式，最好的方式还是按序排列，这样结构上最稳定。

因为${|i - j|\ne 0}$的存在，所以相邻元素不能相等，这也是最低要求。

因此这题的思路，就是排序，然后校验相邻元素是否相等。

如果不存在相等的元数据，则是可行的构造解.


```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            int[] arr = new int[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextInt();
            }
            // 排序
            Arrays.sort(arr);
            // 进行校验
            boolean f = true;
            for (int i = 0; f&&i < arr.length -1 ; i++) {
                if (arr[i] == arr[i + 1]) f = false;
            }
            // 输出结果
            if (!f) System.out.println("-1");
            else {
                System.out.println(Arrays.toString(arr).replaceAll("\\[|\\]|,", ""));
            }
        }
    }

}
```

---
## [D. Kevin喜欢零(简单版本)](https://ac.nowcoder.com/acm/contest/57683/D)

好像第一版就是最优解了，所以放到E中一起讲

---
## [E. Kevin喜欢零(困难版本)](https://ac.nowcoder.com/acm/contest/57683/E)

求$x=\prod_{t=i}^{t=j} a_t$ 中 恰有k个0， 求满足这样连续子区间个数

这种连续子区间，满足某些条件的个数，往往和滑动窗口有关系, 就是固定一端点，然后快速找到另一侧的端点范围，最后累加即可。

当然这题另一个切入点是乘积末尾0的个数，一般是找因子2/5的个数.

所以这题的思路，就是分别维护2/5个数的前缀和，然后滑窗即可。

因为2/5有两个，所以这题的特殊之处是不是传统意义上的双指针，还是三指针做法。

$p1=min(区间和2的个数, 区间和5的个数)==k的左边界$

$p2=min(区间和2的个数, 区间和5的个数)>k的左边界$

那[p1, p2)这一段就是可行解的右侧范围

由于滑窗，p1,p2不会回撤，只会单调增长

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            int k = sc.nextInt();
            
            // 前缀和维护(2/5)
            int[] sum2 = new int[n + 1];
            int[] sum5 = new int[n + 1];

            for (int i = 0; i < n; i++) {
                int v = sc.nextInt();
                int cnt2 = 0, cnt5 = 0;
                while (v % 2 == 0) {
                    v /= 2;
                    cnt2++;
                }
                while (v % 5 == 0) {
                    v /= 5;
                    cnt5++;
                }
                sum2[i + 1] = sum2[i] + cnt2;
                sum5[i + 1] = sum5[i] + cnt5;
            }

            long ans = 0;

            // 三指针做法
            int p1 = 0, p2 = 0;
            // 枚举左端点，固定之然后滑动
            for (int i = 0; i < n; i++) {
                // 一个找到min(2/5)==k的左边界
                while (p1 < i || (p1 < n && Math.min(sum5[p1 + 1] - sum5[i],sum2[p1 + 1] - sum2[i]) < k)) {
                    p1++;
                }
                // 一个找到min(2/5)>k的左边界
                while (p2 < i || (p2 < n && Math.min(sum5[p2 + 1] - sum5[i], sum2[p2 + 1] - sum2[i]) <= k)) {
                    p2++;
                }
                
                if (p1 < n && Math.min(sum5[p1 + 1] - sum5[i],sum2[p1 + 1] - sum2[i]) == k) {
                    ans += (p2 - p1);
                }
            }

            System.out.println(ans);

        }
    }

}
```

这个可能有些绕，或者说一下子转到三指针，有些难受。

那可以退化为枚举+二分(寻找右侧的floor和ceiling边界)，这样可能更顺一些

---

## [F. Kevin的哈希构造](https://ac.nowcoder.com/acm/contest/57683/F)

定义了一个字符串s的hash

> ${hash=(s_1*b^{n-1}+s_2*b^{n-2}+...+s_n*b^0)\% p}$

求另一个字符串t，刚好他们相等的字符恰好为k个，且hash值相等

可以观察下$n(<=50),k(<=50),p(<=1000)$的范围, 是不是可以大概猜到什么？

没错，就是DP构造

可以如下设计状态

${opt[i][j][26][q]}$
- i为字符串第i个字母，
- j为前i项中相等元素有几个，
- 26是字母表，表示第i字符是字母是啥，
- q为前i项的hash前缀和, $q\in [0, p)$

状态转移为26，因为每次可以选择26个字母

但是这样的时间复杂度为，$状态数*转移代价=50*50*26*1000*26$, 这是一个极其可怕的数

能否在压缩下空间，降下维度

其实可以把中间26去掉，因为它属于结果

这样新状态为$opt[i][j][q]$

状态转移方程

$s_{i+1}=buf[i+1] * b^{(n - 1 - i)} \% p$

${opt[i+1][j+1][q+s_{i+1}]\ |= opt[i][j][q]}, 即t[i+1]=s[i+1]$

${opt[i+1][j][q+t_{i+1}]\ |= opt[i][j][q]}, 即t[i+1]\ne s[i+1]$

因为最后还要构造解，所以需要引入一个状态对象，用于追踪回溯前一个状态转移过来(j,q)，以及当前选择的字符c

因此最终的解为 opt[n - 1][k][hash(s)] 是否为true



```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static class Tx {
        int k, p;
        char c;
        public Tx(int k, int p, char c) {
            this.k = k;
            this.p = p;
            this.c = c;
        }
    }

    static String solve(int n, int b, int p, int k, char[] buf) {

        if (n == k) {
            return new String(buf);
        }

        long[] bmi = new long[n];
        bmi[n - 1] = 1;
        for (int i = n - 2; i >= 0; i--) {
            bmi[i] = (bmi[i + 1] * b) % p;
        }

        int h = 0; // 原始字符串的hash值
        for (int i = 0; i < buf.length; i++) {
            h = (int)(h + ((long)(buf[i] - 'a' + 1) * bmi[i] % p)) % p;
        }

        // 取反
        Tx[][][] trace = new Tx[n][k + 1][p];
        boolean[][][] opt = new boolean[n][k + 1][p];
        int first = (int)(buf[0] - 'a' + 1);
        for (int i = 1; i <= 26; i++) {
            char c = (char)(i - 1 + 'a');
            int offset = (int)((i * bmi[0]) % p);
            if (first == i) {
                if (1 <= k) {
                    opt[0][1][offset] = true;
                    trace[0][1][offset] = new Tx(0, offset, c);
                }
            }
            else {
                opt[0][0][offset] = true;
                trace[0][0][offset] = new Tx(0, offset, c);
            }
        }

        for (int i = 0; i < n - 1; i++) {
            int now = (int)(buf[i + 1] - 'a' + 1);
            for (int j = 0; j <= k; j++) {
                for (int t = 0; t < p; t++) {
                    if (opt[i][j][t] == false) continue;
                    for (int s = 1; s <= 26; s++) {
                        char nc = (char)(s - 1 + 'a');
                        int offset = (int)((t + (s * bmi[i + 1]) % p) % p);
                        if (now == s) {
                            if (j + 1 <= k) {
                                opt[i + 1][j + 1][offset] = true;
                                trace[i + 1][j + 1][offset] = new Tx(j, t, nc);
                            }
                        } else {
                            opt[i + 1][j][offset] = true;
                            trace[i + 1][j][offset] = new Tx(j, t, nc);
                        }
                    }
                }
            }
        }

        if (!opt[n - 1][k][h]) return "-1";

        int np = h;
        int nk = k;
        int ni = n - 1;
        StringBuilder sb = new StringBuilder();
        while (ni >= 0) {
            Tx cur = trace[ni][nk][np];
            sb.append(cur.c);
            np = cur.p;
            nk = cur.k;
            ni--;
        }
        return sb.reverse().toString();
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt(), b = sc.nextInt();
            int p = sc.nextInt(), k = sc.nextInt();
            char[] buf = sc.next().toCharArray();
            System.out.println(solve(n, b, p, k, buf));
        }
    }

}
```

--
## [G. MoonLight的冒泡排序难题](https://ac.nowcoder.com/acm/contest/57683/G)

就是求一个排序的swap次数（注意是每个数按序主动swap每行的次数，不是两两交换的swap次数）

然后求任意排列下，期望值？

主要还是推导公式，最大的数为(n-1)/n, 次大的数为(n-2/(n-1), ..., 第二小的数为1/2

则${E=\sum_{i=2}^{i=n} (i-1)/i}$

那这个期望值，按分子，分母迭代分别计算即可，最后套一下逆元（题目描述用了费马小定律）


```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static long mod = 998244353l;
    static int max = 200_000 + 1;
    static long[][] opt = new long[max][2];

    // 离线预处理全部的解
    static void init() {
        opt[1][0] = 0;
        opt[1][1] = 1;
        for (int i = 2; i < max; i++) {
            opt[i][0] = (opt[i - 1][0] * i % mod + (long)(i - 1) * opt[i - 1][1] % mod) % mod;
            opt[i][1] = opt[i - 1][1] * i % mod;
        }
    }

    static long ksm(long b, long v, long mod) {
        long r = 1l;
        while (v > 0) {
            if (v % 2 == 1) {
                r = r * b % mod;
            }
            v >>= 1;
            b = b * b % mod;
        }
        return r;
    }

    public static void main(String[] args) {
        // 预处理优化
        init();

        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        
        while (kase-- > 0) {
            int n = sc.nextInt();
            // 在线查询即可
            long fz = opt[n][0], fm = opt[n][1];
            System.out.println(fz * ksm(fm, mod - 2, mod) % mod);
        }

    }

}

```

---

# 写在最后

人生不仅仅是为了自己而不停向前冲刺，把某人的幸福当成自己的幸福，还有这样的生活方式。

![alt](https://uploadfiles.nowcoder.com/images/20230615/446702330_1686820880717/D2B5CA33BD970F64A6301FA75AE2EB22)


