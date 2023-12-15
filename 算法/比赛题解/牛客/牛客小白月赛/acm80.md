# 牛客小白月赛80 解题报告 | 珂学家 | 前缀和优化的二分 + 二分图最大匹配


---
# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231028/446702330_1698460086386/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价

这场好像比前几场小白月整体要简单。《放学后》系列贯穿3题，突然想起来东野圭吾的《放学后》，现在的故事情节，还历历在目。

E，F挺有意思的，只是仅仅看着像博弈。

---

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 矩阵快速幂签到](https://ac.nowcoder.com/acm/contest/67730/A)

非常优秀的一道题，明示矩阵幂

但是手玩一下，可以发现规律

a(n) = n + 1

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int mod = 998244353;
        System.out.println((n + 1) % mod);
    }
    
}
```
---

## [B. 第一次放学](https://ac.nowcoder.com/acm/contest/67730/B)

贪心，就是找到最大的组的数p

k <= n - p,  则为p

k > n - p, 则为n - k

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), m = sc.nextInt(), k = sc.nextInt();

        Map<Integer, Integer> hash = new HashMap<>();
        for (int i = 0; i < n; i++) {
            int v = sc.nextInt();
            hash.merge(v, 1, Integer::sum);
        }

        // 获取最大的组
        int p = hash.values().stream().mapToInt(Integer::valueOf).max().getAsInt();

        if (k <= n - p) {
            System.out.println(p);
        } else {
            System.out.println(n - k);
        }
    }

}
```


---
## [C. 又放学辣（简单）](https://ac.nowcoder.com/acm/contest/67730/C)

和D一起讲

---

## [D. 又放学辣（进阶）](https://ac.nowcoder.com/acm/contest/67730/D)

大致两种思路

- 在线解法

  二分框架，但是核心check逻辑需要预处理前缀和/后缀和优化加速

- 离线

  需要把查询按大小把顺序打乱，这样就可以类似双指针来优化，大大降低时间复杂度

在线解法，预处理前缀和/后缀和, 二分解法，感觉更容易易读一些.

```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
import java.util.function.BiFunction;
import java.util.stream.Collectors;

public class Main {

    static int solve(int left, int idx, BiFunction<Integer, Integer, Boolean> checker) {
        int l = 0, r = left;
        while (l <= r) {
            int m = l + (r - l) / 2;
            if (checker.apply(m, idx)) {
                r = m - 1;
            } else {
                l = m + 1;
            }
        }
        return l;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt(), m = sc.nextInt(), k = sc.nextInt();
        int[] groups = new int[m + 1];

        for (int i = 0; i < n; i++) {
            int v = sc.nextInt();
            groups[v]++;
        }

        // 这边维护后缀和
        int[] cnt = new int[n + 1];
        int[] suf = new int[n + 1];
        for (int i = 1; i <= m; i++) {
            suf[groups[i]] += groups[i];
            cnt[groups[i]] += 1;
        }
        for (int i = n - 1; i >= 0; i--) {
            suf[i] += suf[i + 1];
            cnt[i] += cnt[i + 1];
        }

        // 定义核心check逻辑
        BiFunction<Integer, Integer, Boolean> checker = (mid, idx) -> {
            // check代码
            long need = suf[mid] - cnt[mid] * mid;
            if (groups[idx] >= mid) {
                need -= (groups[idx] - mid);
            }
            return need <= k;
        };

        List<Integer> res = new ArrayList<>();
        for (int i = 1; i <= m; i++) {
            if (n - groups[i] < k) {
                res.add(-1);
            } else {
                res.add(solve(n - groups[i], i, checker));
            }
        }
        System.out.println(res.stream().map(String::valueOf).collect(Collectors.joining(" ")));
    }
    
}
```

---

## [E. 一种因子游戏](https://ac.nowcoder.com/acm/contest/67730/E)

二分图最大匹配模板

也想不到其他的一些解法，没法有效证明，感觉这题出的太板了。

```java
import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

public class Main {

    // 无权二分图最大匹配算法 O(VE)
    static boolean match(int u, int[] link, boolean[] used, List<Integer>[]g) {
        for (int v: g[u]) {
            if (used[v]) continue;
            used[v] = true;
            if (link[v] == 0 || match(link[v], link, used, g)) {
                link[u] = v;
                link[v] = u;
                return true;
            }
        }
        return false;
    }

    static int gcd(int a, int b) {
        return b == 0 ? a : gcd(b, a % b);
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        int[] brr = new int[n];

        List<Integer>[]g = new List[2 * n];
        Arrays.setAll(g, x -> new ArrayList<>());

        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            brr[i] = sc.nextInt();
        }

        boolean ok = true;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (gcd(arr[i], brr[j]) == 1) {
                    g[i].add(j + n);
                    g[j + n].add(i);
                }
            }
            if (g[i].size() == 0) {
                ok = false;
                break;
            }
        }

        if (!ok) {
            System.out.println("Alice");
            return;
        }

        int ans = 0;
        int[] link = new int[2 * n];
        for (int i = n; i < 2 * n; i++) {
            if (link[i] != 0) continue;
            boolean[] used = new boolean[2 * n];
            used[i] = true;
            if (match(i, link, used, g)) {
                // 找到一条增广路径
                ans++;
            }
        }

        System.out.println(ans < n ? "Alice" : "Bob");
    }

}
```

时间复杂度为O(EV) = O(n^3)

---

## [F. 一种异或游戏](https://ac.nowcoder.com/acm/contest/67730/F)

n为Alice的牌数，m为Bob的牌数

先引入几个概念

- 被限定的数，特指 Alice牌中的数，且能在 Bob牌中找到 异或K

- 限定的数，特指 Bob牌中的数，能挤兑Alice牌，构建异或K

我们需要的是

- 被限定的数的总个数，注意是总个数 Hits

- 限定的数的种类数，注意是种类数 Category

----
```
举例：

Alice:  2 2 2 4 3 5

Bob:    2 2 11

其中被限定总个数为4, 分别为 2, 2, 2, 4

限定数为1个，即2(去重)
```
---

再理解以上这几个概念后，后面的分类讨论，基本就这几个概念完成的

- 对于 n <= m

  如果 Hits > 1, 则Alice必输
  反之 Alice一定赢，先出完牌也算赢

- n > m

  这边还需要在细分下

    1. n - Hits + 1 >= m, Alice必赢

    2. n - Hits + 1 < m

       m - (n - Hits) < Category, Alice必赢，因为Bob必输

       m - (n - Hits) >= Category, Bob必赢，Alice输了


```java
import java.io.*;
import java.util.*;

public class F {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int m = sc.nextInt();
        int k = sc.nextInt();

        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        int[] brr = new int[m];
        for (int i = 0; i < m; i++) {
            brr[i] = sc.nextInt();
        }
        int maxAlice = Arrays.stream(arr).max().getAsInt();
        int maxBob = Arrays.stream(brr).max().getAsInt();

        // 得用数组hash，替代java容器hashmap，卡常
        int[] alice = new int[maxAlice + 1];
        int[] bob = new int[maxBob + 1];
        for (int i = 0; i < n; i++) {
            alice[arr[i]]++;
        }
        for (int i = 0; i < m; i++) {
            bob[brr[i]]++;
        }

        // --------------------------------------

        // 下面是核心代码
        // 计算被限制的数，以及限制数的种类
        int category = 0;
        int hits = 0;
        for (int i = 0; i <= maxAlice; i++) {
            if (alice[i] > 0 && (i ^ k) <= maxBob && bob[i ^ k] > 0) {
                hits += alice[i];
                category++;
            }
        }

        // 胜负规则
        if (n <= m) {
            if (hits > 1) {
                System.out.println("Bob");
            } else {
                System.out.println("Alice");
            }
        } else {
            if (n - hits + 1 >= m) {
                System.out.println("Alice");
            } else {
                if (m - (n - hits) < category) {
                    System.out.println("Alice");
                } else {
                    System.out.println("Bob");
                }
            }
        }

    }

}

```



---

# 写在最后
![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231028/446702330_1698460364395/D2B5CA33BD970F64A6301FA75AE2EB22)





