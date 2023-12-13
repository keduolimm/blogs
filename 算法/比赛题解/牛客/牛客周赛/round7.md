# 牛客周赛 Round 7 解题报告 | 珂学家 | 状态机DP + 数学场

---

# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230813/446702330_1691933116422/D2B5CA33BD970F64A6301FA75AE2EB22)

所有他努力长大成人的日子，不过是为了与她相遇。

---

# 整体评价

这场C有点考验智商，太难了，最后用了暴力解，挺虚了的，感觉数据弱了。D到是一眼题，套路满满，反而简单，应该是属于状态计数DP。

---

## [A. 游游的you矩阵](https://ac.nowcoder.com/acm/contest/63091/A)

就是把字符转换为$2^i$的幂次，这样方便统计.
> y -> 1, o -> 2, u -> 4, 其他字符皆为0

这样2x2的矩阵，如果其按或和为7，则为满足矩阵

### 常规解法

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int h = sc.nextInt(), w = sc.nextInt();

        int[][] states = new int[h][w];
        char[][] grid = new char[h][];
        for (int i = 0; i < h; i++) {
            grid[i] = sc.next().toCharArray();
            for (int j = 0; j < w; j++) {
                if ("you".indexOf(grid[i][j]) != -1) {
                    states[i][j] = 1 << ("you".indexOf(grid[i][j]));
                }
            }
        }
        int ans = 0;
        for (int i = 0; i < h - 1; i++) {
            for (int j = 0; j < w - 1; j++) {
                int state = states[i][j] | states[i + 1][j] | states[i][j + 1] | states[i + 1][j + 1];
                if (state == 7) {
                    ans++;
                }
            }
        }
        System.out.println(ans);
    }

}
```

---

### 题外话

如果这题的区域不是$2\times2$, 而是更大的长宽，这个该如何求解呢？

如果按照枚举起点，那时间复杂度为升级为 $O(h^2\times w^2)$, h,w为矩阵长宽，这是没法接受的。

我记得abc上有类似的题，我当时的思路是“Z”字形移动子区域，然后动态维护子区域内的计数.

这样每个格子进入一次，进出一次，均摊为$O(1)$

因此时间复杂度为$O(h\times w)$


---

## [B. 游游的01串操作](https://ac.nowcoder.com/acm/contest/63091/B)

对于01串，非常具有CF风格的题，不过这边01是交替出现的，这就相对容易了。

可以枚举模拟，也可以按照以往的套路状态机DP求解。

### 1. 状态机DP

很简单的状态机DP

令$opt[i][s]$, 表示前i个元素，以s(0/1)结尾的最小代价

${
\left\{\begin{matrix}
opt[i][0]=opt[i - 1][1] + i*(str[i] - '0') \\
opt[i][1]=opt[i - 1][0] + i*('1' - str[i])
\end{matrix}\right.
}$

$i$只从$i-1$转移过来，因此可以利用滚动数组进行优化，而s的维度很小，只有0/1两个状态，因此可以最简化为2个变量。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();

        long s0 = 0, s1 = 0;
        for (int i = 0; i < str.length; i++) {
            char c = str[i];
            long t0 = 0, t1 = 0;
            if (c == '1') {
                t0 = (long)(i + 1) + s1;
                t1 = s0;
            } else {
                t0 = s1;
                t1 = (long)(i + 1) + s0;
            }
            s0 = t0;
            s1 = t1;
        }
        System.out.println(Math.min(s0, s1));
    }
    
}
```

---
### 2. 枚举模拟

这题只有两种可能，即'1'开头，’0‘开头

那么枚举模拟就可，然后取最小值

```java
import java.io.BufferedInputStream;
import java.util.Scanner;
 
public class Main {
    
    static long solve(char[] str, char s) {
        long cost = 0;
        for (int i = 0; i < str.length; i++) {
            if (s != str[i]) cost += (i + 1);
            s = (s == '1') ? '0' : '1';
        }
        return cost;
    }
 
    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        System.out.println(
            Math.min(solve(str, '1'), solve(str, '0'))
        );
    }
     
}
```


---

## [C. 游游的正整数](https://ac.nowcoder.com/acm/contest/63091/C)

没有找到规律，所以一开始试了二分(结果wa了，后来才知道判不存在的条件变小了)，因此试想是不是只能枚举求解。

怀着忐忑不安的心情，试了枚举的思路。

即通过找到上下界，在上界中从大从小找最大值，在下界中从小到大找最小值。

感觉数据有些弱，竟然过了。

### 1. 枚举

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            long a = sc.nextInt(), b = sc.nextInt();
            long l = sc.nextInt(), r = sc.nextInt();

            long maxCi = -1, minCi = -1;
            long ti = ((b - a) + (l - 1)) / l;

            // 方向很重要
            for (long i = ti; i * r >= (b - a); i--) {
                long sum = i * r;
                long diff = sum - (b - a);
                if (diff <= i * (r - l)) {
                    maxCi = i;
                    break;
                }
            }

            long tj = ((b - a) + (r - 1)) / r;
            for (long j = tj; j * l <= (b - a); j++) {
                long sum = j * r;
                long diff = sum - (b - a);
                if (diff <= j * (r - l)) {
                    minCi = j;
                    break;
                }
            }

            if (maxCi == -1) System.out.println(-1);
            else System.out.println(minCi + " " + maxCi);

        }

    }

}
```

### 2. 二分求解

这题有个特殊，就是最大值容易求，最小值不太确定.

而且存在阶段性，因此可用二分求解。

注意，不存在情况为

$(b-a) \% l > (r-l) * ((b-a)/l)$

即按最大组构造$t=(b-a)/l$，剩余的数$re=(b-a)\%l$ 超过了 $t * (r - l)$, 这种情况没法构造，那更小的组，自然就没法构造了，所以一定不存在。

除此之外，既然最大组存在了，那最小组就有兜底解了(就是和最大组一致).


```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            long a = sc.nextInt(), b = sc.nextInt();
            long l = sc.nextInt(), r = sc.nextInt();

            // k*l + [0,..., r-l] * k
            long d = b - a;
            long t = d / l;
            long re = d % l;

            if (re > t * (r - l)) {
                // 这是无解的情况
                System.out.println(-1);
            } else {
                // 最大的就是t了
                // 最少的次数呢？
                long maxCi = t;
                long minCi = t;

                // 存在阶段性
                long left = 0, right = maxCi;
                while (left <= right) {
                    long mid = left + (right - left) / 2;
                    if (mid * r >= d && mid * r - d <= (r - l) * mid) {
                        minCi = mid;
                        right = mid - 1;
                    } else {
                        left = mid + 1;
                    }
                }
                System.out.println(minCi + " " + maxCi);
            }
        }
    }

}
```

---

## [D. 游游的选数乘积](https://ac.nowcoder.com/acm/contest/63091/D)

非常经典的状态计数DP

因为1e9范围内，最多13个5,30个2

因此设计为 $opt[14][31]$, 第一维度表示5的因子个数，第二维度表示2的因子个数

那结果为

$s1 = \sum_{i=0}^{13}\sum_{j=0}^{30} opt[i][j] * (opt[i][j] - 1) / 2$


$s2=\sum_{i=0}^{13}\sum_{j=0}^{30}\sum_{s=0}^{13}\sum_{t=0}^{30}{opt[i][j] * opt[s][t]}, (i, j) \ne (s, t)$

最终结果为$s=s1+s2/2$, $s2$刚好重复计算2次

### 1. 经典状态计数DP

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static int[] split(int v) {
        int[] res = new int[] {0, 0};
        while (v % 2 == 0) {
            v /= 2;
            res[0]++;
        }
        while (v % 5 == 0) {
            v /= 5;
            res[1]++;
        }
        return res;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), x = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        long[][] opt = new long[14][31];
        for (int i = 0; i < n; i++) {
            int[] xx = split(arr[i]);
            opt[xx[1]][xx[0]]++;
        }

        long ans1 = 0, ans2 = 0;
        for (int i = 0; i < 14; i++) {
            for (int j = 0; j < 31; j++) {
                if (opt[i][j] == 0) continue;
                for (int h = 0; h < 14; h++) {
                    for (int w = 0; w < 31; w++) {
                        if (opt[i][j] == 0) continue;
                        if (i + h < x || j + w < x) continue;
                        if (i == h && j == w) {
                            ans1 += opt[i][j] * (opt[i][j] - 1) / 2;
                        } else {
                            ans2 += opt[i][j] * opt[h][w];
                        }
                    }
                }
            }
        }

        System.out.println(ans1 + ans2 / 2);
    }

}
```


### 2. 二维偏序写法

这是一类二维偏序的题，参考了某大神的写法。

一般是固定一个维度(变成0-1可见问题)，然后另一个维度借助树状数组/线段树来范围查询。

这边的树状数组, 引入带偏移(因为树状数组内部从1开始计数), 但是外部的语义可以保持一致,比如从0，甚至-1开始统计.

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    // 支持索引从0开始的树状数组(带偏移)
    static class BIT {
        int n;
        int[] arr;

        int offset;

        public BIT(int n, int offset) {
            this.offset = offset;
            this.n = n + offset;
            this.arr = new int[n + offset + 1];
        }

        void update(int p, int d) {
            p += offset;
            while (p <= n) {
                this.arr[p] += d;
                p += p & -p;
            }
        }

        int query(int p) {
            p += offset;
            int res = 0;
            while (p > 0) {
                res += arr[p];
                p -= p & -p;
            }
            return res;
        }
    }

    static class Tx {
        int i2, i5;
        public Tx(int i2, int i5) {
            this.i2 = i2;
            this.i5 = i5;
        }
    }

    static int split(int v, int d) {
        int res = 0;
        while (v % d == 0) {
            v /= d;
            res++;
        }
        return res;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), x = sc.nextInt();

        Tx[] arr = new Tx[n];
        for (int i = 0; i < n; i++) {
            int v = sc.nextInt();
            arr[i] = new Tx(split(v, 2), split(v, 5));
        }

        long ans = 0;
        // 二维偏序的写法
        //      以5因子为第一维度
        Arrays.sort(arr, Comparator.comparingInt(t -> t.i5));

        //      存放2因子的数的树状数组
        BIT bit = new BIT(32, 2);
        //      栈，用于缓存
        Deque<Tx> deq = new ArrayDeque<>();
        for (int i = 0; i < n; i++) {
            Tx cur = arr[i];
            while (!deq.isEmpty() && deq.peekLast().i5 + cur.i5 >= x) {
                Tx item = deq.pollLast();
                bit.update(item.i2, 1);
            }

            ans += bit.query(32) - bit.query(Math.max(0, x - cur.i2) - 1);
            deq.addLast(cur);
        }

        System.out.println(ans);

    }

}
```

---

# 写在最后

放心，我不会被吹走的！

![alt](https://uploadfiles.nowcoder.com/images/20230813/446702330_1691933151959/D2B5CA33BD970F64A6301FA75AE2EB22)
