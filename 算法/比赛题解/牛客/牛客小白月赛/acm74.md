# 牛客小白月赛74 解题报告 | 珂学家 | 二分+贪心+单调栈

---
# 前言

![alt](https://uploadfiles.nowcoder.com/images/20230610/446702330_1686355490741/D2B5CA33BD970F64A6301FA75AE2EB22)

雪的碗里，盛的是月光。

---
# 整体评价

题目质量出的挺好的，可以一题多解，而且覆盖面也广，赞一个。

F题除了经典的二分，感觉直接贪心(类似最小生成树）也可以，而且是否可以借助可撤销的并查集来常数级优化，本文将给出解答。

G题是一道经典的单调栈优化的题，本质还是贪心，而且从思路上看和D题有一定的渊源。

--- 

## [A. 简单的整除](https://ac.nowcoder.com/acm/contest/59284/A)

签到题，不细说了

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class A {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        if (n % 2 == 0 || n % 3 == 0 || n % 5 == 0 || n % 7 == 0) {
            System.out.println("YES");
        } else {
            System.out.println("NO");
        }
    }

}
```
---

## [B. 整数划分](https://ac.nowcoder.com/acm/contest/59284/B)

有意思的题，不过这里的字典序是自然序的意思

那就是一路贪心，知道没法生成连续的自然数为止

```java
import java.io.BufferedInputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();

            List<Integer> res = new ArrayList<>();
            for (int i = 1; n - i > i; i++) {
                n -= i;
                res.add(i);
            }
            res.add(n);

            System.out.println(
                    res.stream().map(x -> String.valueOf(x)).collect(Collectors.joining(" "))
            );
        }
    }

}
```

---

## [C. 传送阵](https://ac.nowcoder.com/acm/contest/59284/C)

在同值无代价跳转，非同值，只能上下左右移动

看似是一个很复杂的问题，其实本质就是寻找不同值的个数

很好的一道思维题

```
import java.io.BufferedInputStream;
import java.util.Scanner;
import java.util.TreeSet;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int h = sc.nextInt();
            int w = sc.nextInt();

            TreeSet<Integer> st = new TreeSet<>();
            for (int i = 0; i < h * w; i++) {
                st.add(sc.nextInt());
            }
            System.out.println(st.size());
        }
    }
    
}
```

---

## [D. 修改后的和](https://ac.nowcoder.com/acm/contest/59284/D)

经典的贪心问题

对于每个非负数的点，独立进行收益计算

然后排序（从大到小）进行最多m次贪心，这样求解下来的值，永远是最大的。

实际上贪心挑选的点集${S(a_i, a_j, a_k...)}$，并不影响实际的执行顺序，执行顺序一定还是倒序来执行的，即点集S中，最右边的一定先执行，最左边的最后执行。

这也是这道思维题，有点绕的地方.

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt(), m = sc.nextInt();
            long[] arr = new long[n];
            long acc = 0;
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextLong();
                acc += arr[i];
            }
            
            // 挑选m个元素
            List<Long> lists = new ArrayList<>();
            for (int i = 0; i < n; i++) {
                if (arr[i] >= 0) {
                    lists.add(1l *arr[i] * (n - i));
                }
            }

            Collections.sort(lists, Comparator.comparing(x -> -x));

            for (int i = 0; i < m && i < lists.size(); i++) {
                acc -= lists.get(i);
            }
            System.out.println(acc);

        }
    }

}
```

---

## [E. 幼稚园的树2](https://ac.nowcoder.com/acm/contest/59284/E)

数据模拟题，需要分类讨论

这里面还有一个周期的概念，用于加速收敛

- 不需要剪枝，一直生长
- 需要剪枝，则进入一个特定生长周期

```java
import java.io.BufferedInputStream;
import java.util.Scanner;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Main {

    static long solve(long h, long inc, long m, long k, long b) {

        if (h + inc * (m - 1) <= k) return h + inc * (m - 1);

        long days = (k - h) / inc + 1;
        long left = m - 1 - days;
        if (left == 0) return b;

        // 这个周期很重要
        long round = (k - b + inc) / inc;
        left %= round;

        return b + left * inc;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            int m = sc.nextInt();
            int k = sc.nextInt();
            int b = sc.nextInt();

            int[] hs = new int[n];
            int[] inc = new int[n];

            for (int i = 0; i < n; i++) {
                hs[i] = sc.nextInt();
            }
            for (int i = 0; i < n; i++) {
                inc[i] = sc.nextInt();
            }

            System.out.println(
                    IntStream.range(0, n).mapToObj(i -> String.valueOf(solve(hs[i], inc[i], m, k,b)))
                    .collect(Collectors.joining(" "))
            );
        }
    }

}
```

--- 
## [F. 最便宜的构建](https://ac.nowcoder.com/acm/contest/59284/F)

很典的题，求满足连通图集合的前提下，满足的所有边集合S下，最大的边最小值是什么？

一般这种最大最小的问题，就用二分的思路。

因为是连通图校验，所以最自然的方式是，二分+并查集

### 方法一：二分+并查集

```java
import java.io.BufferedInputStream;
import java.util.*;


public class Main {

    static class Dsu {
        int n;
        int[] arr;
        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
        }

        int find(int u) {
            if (arr[u] == 0) return u;
            return arr[u] = find(arr[u]);
        }

        void union(int u, int v) {
            int ai = find(u);
            int bi = find(v);
            if (ai != bi) {
                arr[ai] = bi;
            }
        }
    }

    static class Solution {

        boolean check(int n, int[][] edges, int mid, int[][] sets) {
            Dsu dsu = new Dsu(n);

            for (int[] e: edges) {
                if (e[2] > mid) break;
                dsu.union(e[0], e[1]);
            }

            for (int[] set: sets) {
                for (int j = 1; j < set.length;j++) {
                    if (dsu.find(set[0]) != dsu.find(set[j])) return false;
                }
            }

            return true;
        }

        int solve(int n, int[][] edges, int[][] sets) {
            Arrays.sort(edges, Comparator.comparing(x -> x[2]));

            int m = edges.length;
            int l = 0, r = m - 1;

            int ans = 0;
            while (l <= r) {
                int mid = l + (r - l) / 2;
                if (check(n, edges, edges[mid][2], sets)) {
                    ans = edges[mid][2];
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            }
            return ans;
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int m = sc.nextInt();

        int[][] edges = new int[m][3];
        for (int i = 0; i < m; i++) {
            edges[i][0] = sc.nextInt();
            edges[i][1] = sc.nextInt();
            edges[i][2] = sc.nextInt();
        }

        int s = sc.nextInt();
        int[][] sets = new int[s][];
        for (int i = 0; i < s; i++) {
            int z = sc.nextInt();
            sets[i] = new int[z];
            for (int j = 0; j < z; j++) {
                sets[i][j] = sc.nextInt();
            }
        }

        Solution solution = new Solution();
        System.out.println(solution.solve(n, edges, sets));

    }

}

```

### 方法二：类最小生成树解法

是否可以利用最小生成树的方法来求解呢？

其实是可以的，这里面核心就是 验证连通性集合 这个能否优化

如果每加一条边，就验证一个集合所有元素，那这个成本/代价是在太高了.

但是这里面其实存在技巧

1. 同一个集合的验证，不需要验证所有的点关系($O(n^2)$)，只要选择一个基准点（比如第0号元素），然后判定剩下的点（$O(n)$)

2. 如果前面的点已经判定OK了，那不需要重判定了

这样这个获选集的判定，是递增的，每一个关系判定均摊是O（1）的操作

这样总代价为 $O(\sum_{i=0}^{i=n} len(sets_i))$


实际实现中，会引入下标对(a, b)，对应sets集合的具体位子

所以贪心解，可能更快速

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Comparator;
import java.util.Scanner;

public class Main {

    static class Dsu {
        int n;
        int[] arr;
        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
        }

        int find(int u) {
            if (arr[u] == 0) return u;
            return arr[u] = find(arr[u]);
        }

        void union(int u, int v) {
            int ai = find(u);
            int bi = find(v);
            if (ai != bi) {
                arr[ai] = bi;
            }
        }
    }

    static class Solution {

        int solve(int n, int[][] edges, int[][] sets) {

            Arrays.sort(edges, Comparator.comparing(x -> x[2]));

            // 维护验证集合的下标 (a, b)
            // a为第a个集合
            // b为第a个集合的第(0, b)pair关系，是否满足连通性
            // 这个(a, b)是递增的，而且前面的一旦满足，不需要再重复判定
            // 一旦a >= len(sets), 就是满足的最小最大值
            int a = 0, b = 1;

            Dsu dsu = new Dsu(n);
            for (int[] e: edges) {
                int u = e[0], v = e[1], w = e[2];
                dsu.union(u, v);

                // 就是这一部分核心代码，看似while，则是是均摊O(1)的操作，理解就能懂了
                while (a < sets.length) {
                    if (b >= sets[a].length) {
                        a++;
                        b = 1;
                    } else if (dsu.find(sets[a][0]) == dsu.find(sets[a][b])) {
                        b++;
                    } else {
                        break;
                    }
                }
                if (a >= sets.length) return w;
            }

            // unreachable
            return -1;

        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int m = sc.nextInt();

        int[][] edges = new int[m][3];
        for (int i = 0; i < m; i++) {
            edges[i][0] = sc.nextInt();
            edges[i][1] = sc.nextInt();
            edges[i][2] = sc.nextInt();
        }

        int s = sc.nextInt();
        int[][] sets = new int[s][];
        for (int i = 0; i < s; i++) {
            int z = sc.nextInt();
            sets[i] = new int[z];
            for (int j = 0; j < z; j++) {
                sets[i][j] = sc.nextInt();
            }
        }

        Solution solution = new Solution();
        System.out.println(solution.solve(n, edges, sets));
    }


}
```


---

## [G. 跳石头，搭梯子](https://ac.nowcoder.com/acm/contest/59284/G)

一道非常有趣的题

这里面有几个有趣的发现

- 如果两个桥是包含关系，那么外面的桥一定优于里面的桥
- 不存在两个桥存在区间重叠（不包含端点），因为违反了桥的定义(即两端点>中间任意的点)
> 比如${B_1(a, c)和B_2(b, d)重叠，那么B1=>c>b, B2=>b>c,从而引发矛盾c>b\ and\ b>c}$

这两个特性非常的重要，决定了这题的最终思路。


这里寻找桥的过程，可以借助单调栈来实现，但这个单调栈会产生存在包含关系的桥，和不存在重叠的桥（不包含端点）。

然后利用排序贪心，利用特性1，先找到最大的桥（最外面），然后过滤掉里面的桥即可。

这样挑选不超过M个的桥，就是最终的答案。

这里面还需要借助前缀和来加速，同时切记 不包含端点。

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    static class Tx {
        long v;
        int s, e;
        public Tx(long v, int s, int e) {
            this.v = v;
            this.s = s;
            this.e = e;
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int m = sc.nextInt();


        List<Tx> cads = new ArrayList<>();

        long[] pre = new long[n];
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            if (i > 0) pre[i] = pre[i - 1] + Math.abs(arr[i] - arr[i - 1]);
        }

        // 单调栈的操作
        Deque<int[]> deq = new ArrayDeque<>();
        for (int i = 0; i < n; i++) {
            int v = arr[i];

            int[] last = null;
            while (!deq.isEmpty() && deq.peekLast()[1] < v) {
                last = deq.pollLast();
            }
            if (!deq.isEmpty()) {
                last = deq.peekLast();
            }

            if (last != null) {
                // 获取一个候选的桥 区间
                long rx = pre[i] - pre[last[0]] - Math.abs(v - last[1]);
                cads.add(new Tx(rx, last[0], i));
            }

            deq.offer(new int[] {i, v});
        }

        // 贪心排序
        cads.sort(Comparator.comparing(x -> -x.v));

        // 利用treeset来实现，区间重叠的判定
        // 即[s, e]只要有一个点被包含，即存在外面更大的桥（包含它），忽略即可
        TreeSet<Integer> set = new TreeSet<>();
        long ans = pre[n - 1];
        for (Tx cur: cads) {
            if (m <= 0) break;

            int s = cur.s, e = cur.e;

            Integer hk = set.ceiling(s);
            if (hk != null && hk <= e) {
                continue;
            }
            // 需要去掉端点
            for (int i = s + 1; i < e; i++) {
                set.add(i);
            }

            m--;
            ans -= cur.v;
        }

        System.out.println(ans);
    }

}
```

---


# 写在最后

此心想念你

碎成千片

我一片也不丢

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230610/446702330_1686355663114/D2B5CA33BD970F64A6301FA75AE2EB22)



