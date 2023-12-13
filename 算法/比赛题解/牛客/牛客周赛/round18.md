# 牛客周赛 Round 18 解题报告 | 珂学家 | 分类讨论计数 + 状态DP

---
# 前言

![alt](https://uploadfiles.nowcoder.com/images/20231106/446702330_1699204226113/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价

前三题蛮简单的，T4是一个带状态的DP，这题如果用背包思路去解，不知道如何搞，感觉有点头痛。所以最后还是选择状态DP来求解。

---

欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---
## [A. 游游的整数翻转](https://ac.nowcoder.com/acm/contest/68346/A)

这题最好是用API来处理，这样更简洁且准确率高

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        String s = sc.next();
        int k = sc.nextInt();

        String s1 = s.substring(0, k), s2 = s.substring(k);
        Long r = Long.valueOf(
                new StringBuilder(s1).reverse()
                        .append(s2).toString()
        );
        System.out.println(r);
    }

}

```

---

## [B. 游游的排列统计](https://ac.nowcoder.com/acm/contest/68346/B)

很有意思的一道题

这题甚至可以用状压DP求解

不过这边还是用暴力dfs, 其一，n<10, 其二，约束强，剪枝大

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static boolean[] primes = new boolean[20];

    static {
        primes[2] = true;
        primes[3] = true;
        primes[5] = true;
        primes[7] = true;
        primes[11] = true;
        primes[13] = true;
        primes[17] = true;
        primes[19] = true;
    }

    static long dfs(int n, int u, int s) {
        if (s == ((1 << n) - 1)) {
            return 1;
        } else {
            long res = 0;
            for (int i = 1; i <= n; i++) {
                if (((1 << (i - 1)) & s) != 0) continue;
                if (primes[u + i]) continue;
                res += dfs(n, i, s | (1 << (i - 1)));
            }
            return res;
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        long res = 0;
        for (int i = 1; i <= n; i++) {
            res += dfs(n, i, 1 << (i - 1));
        }

        System.out.println(res);
    }

}
```

---

## [C. 游游刷题](https://ac.nowcoder.com/acm/contest/68346/C)

同余分组计数

然后贪心分类讨论

- r = 0, 则单独为一组 cnt(0)
- 2 * r = k, 则为 cnt(r) / 2
- r < k,  则 min(cnt(r), cnt(k - r))

累加即可

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int k = sc.nextInt();
        Map<Integer, Integer> hash = new HashMap<>();
        for (int i = 0; i < n; i++) {
            int v = sc.nextInt();
            hash.merge(v % k, 1, Integer::sum);
        }

        long ans = 0;
        for (Map.Entry<Integer, Integer> kv : hash.entrySet()) {
            int k1 = kv.getKey(), v1 = kv.getValue();
            if (k1 == 0) {
                ans += v1;
            } else if (k1 * 2 == k){
                ans += v1 / 2;
            } else if (k1 * 2 < k) {
                int v2 = hash.getOrDefault(k - k1, 0);
                ans += Math.min(v2, v1);
            }
        }
        System.out.println(ans);
    }

}
```

---

## [D. 游游买商品](https://ac.nowcoder.com/acm/contest/68346/D)

状态DP

令 dp[i][j][s],  i为前i个项，j表示使用了多少钱，s为0,1表示状态

> 0表示 第i项不购买，或者半价购买

> 1表示 第i项全价购买

那状态转移为

> dp[i][j][0] = max(dp[i - 1][j][0], dp[i - 1][j][1], dp[i - 1][j - cost[i] / 2][1] + happy[i]);

> dp[i][j][1] = max(dp[i - 1][j - cost[i]][0], dp[i - 1][j - cost[i]][1]) + happy[i];


最后的结果为 max(dp[n - 1][j][s]),  0<=j<=x, s=0,1

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), x = sc.nextInt();
        int[] cost = new int[n];
        long[] happy = new long[n];
        for (int i = 0; i < n; i++) {
            cost[i] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            happy[i] = sc.nextLong();
        }

        // *)
        long inf = Long.MIN_VALUE / 10;
        long[][][] opt = new long[n][x + 1][2];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j <= x; j++) {
                opt[i][j][0] = opt[i][j][1] = inf;
            }
        }

        long ans = 0;
        if (cost[0] <= x) {
            opt[0][cost[0]][1] = happy[0];
        }
        opt[0][0][0] = 0;

        for (int i = 1; i < n; i++) {
            for (int j = 0; j <= x; j++) {
                opt[i][j][0] = Math.max(opt[i - 1][j][0], opt[i - 1][j][1]);
                if (j - cost[i] / 2 >= 0) {
                    opt[i][j][0] = Math.max(opt[i][j][0], opt[i - 1][j - cost[i] / 2][1] + happy[i]);
                }
                if (j - cost[i] >= 0) {
                    opt[i][j][1] = Math.max(opt[i - 1][j - cost[i]][0], opt[i - 1][j - cost[i]][1]) + happy[i];
                }
            }
        }
        for (int j = 0; j <= x; j++) {
            ans = Math.max(ans, opt[n - 1][j][0]);
            ans = Math.max(ans, opt[n - 1][j][1]);
        }

        System.out.println(ans);
    }

}
```
----

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20231106/446702330_1699204234319/D2B5CA33BD970F64A6301FA75AE2EB22)





