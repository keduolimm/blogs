
# 牛客周赛 Round 24 解题报告 | 珂学家 | 二分

---
# 前言

这场感觉偏简单，T1构造，T2是求因子的变体题, T3是贪心思维题, T4是很典的二分题。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 小红的矩阵构造](https://ac.nowcoder.com/acm/contest/71993/A)

思路: 构造

按斜对角线切割，并进行构造

对于a(i, j)点，其斜率为i+j

令i+j为奇数，则赋予奇数，反之填冲偶数

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[][] grid = new int[n][n];
        int p1 = 1, p2 = 2;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                int s = i + j;
                if (s % 2 == 0) {
                    grid[i][j] = p1;
                    p1 += 2;
                } else {
                    grid[i][j] = p2;
                    p2 += 2;
                }
            }
        }
        for (int i = 0; i < n; i++) {
            System.out.println(Arrays.stream(grid[i]).mapToObj(String::valueOf).collect(Collectors.joining(" ")));
        }
    }

}
```

---
## [B. 小红的因子](https://ac.nowcoder.com/acm/contest/71993/B)

思路: 因子拆解

从因子v从sqrt(n)枚举到1，这样n/v因子的平方，一定大于n

整体的时间复杂度为$O(nlogn)$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int t = sc.nextInt();
        while (t-- > 0) {
            long n = sc.nextLong();
            long v = 0;
            // 枚举小的因子, sqrt
            for (long i = (int)Math.sqrt(n); i >= 1; i--) {
                // 保证 n/i * n/i > n 等价于 n > i * i
                if (n % i == 0 && n > i * i) {
                    v = n / i;
                    break;
                }
            }
            System.out.println(v);
        }
    }

}
```
---

## [C. 小红的小数点串](https://ac.nowcoder.com/acm/contest/71993/C)

思路: 贪心

- 第一个为 左侧全部，右侧保留一位
- 中间项为左侧尽量大，右侧只保留一位
- 最后一个为 右侧尽量大(不包含第一位)

```java
import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;
import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        String s = sc.next();

        // 根据'.'进行划分
        String[] nums = s.split("\\.");

        // 贪心
        // a0a1.b0b1.c0c1c2.d0d1d2
        // a0a1.b0 b1.c0 c1c2.d0 d1d2

        // 第一个为 左侧全部，右侧保留一位
        // 中间项为左侧尽量大，右侧只保留一位
        // 最后一个为 右侧尽量大(不包含第一位)
        double res = 0;
        int n = nums.length;
        for (int i = 1; i < n - 1; i++) {
            String tv = nums[i].substring(1) + "." + nums[i + 1].charAt(0);
            res += Double.valueOf(tv);
        }
        if (n == 1) {
            res += Double.valueOf(nums[0]);
        } else {
            res += Double.valueOf(nums[0] + "." + nums[1].charAt(0));
        }

        if (n > 1) {
            res += Double.valueOf(nums[n - 1].substring(1));
        }

        System.out.println(res);
    }

}
```
---

## [D. 小红的数组操作](https://ac.nowcoder.com/acm/contest/71993/D)

思路: 二分验证

二分最小的最大值即可

```java
import java.io.*;
import java.util.*;
import java.util.function.Function;

public class Main {

    public static void main(String[] args) {

        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt(), k = sc.nextInt(), x = sc.nextInt();
        long[] arr = new long[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
        }

        Function<Long, Boolean> checker = (m) -> {
            long acc = 0;
            for (int i = 0; i < n; i++) {
                if (arr[i] > m) {
                    long dk = (arr[i] - m + x - 1) / x;
                    acc += dk;
                    if (acc > k) return false;
                }
            }
            return true;
        };

        long minV = Arrays.stream(arr).min().getAsLong();
        long maxV = Arrays.stream(arr).max().getAsLong();
        
        // 设定上下界
        long l = minV - (long)k * x;
        long r = maxV;
        while (l <= r) {
            long m = l + (r - l) / 2;
            if (checker.apply(m)) {
                r = m - 1;
            } else {
                l = m + 1;
            }
        }
        System.out.println(l);

    }

}
```
---

# 写在最后

