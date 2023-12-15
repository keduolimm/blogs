# 牛客小白月赛72 题解报告 | 珂学家 | 二分套二分


---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20230513/446702330_1683942893696/D2B5CA33BD970F64A6301FA75AE2EB22)

“我们的梦想”才不无趣！因为！因为是我们一起创造的！怎么可能输给你这种人！绝对！绝对！绝对不会输！


---

# 整体评价

挺有意思的比赛，感觉难度分布均衡。前三题相对简单，D是动态规划，E是经典二分题，F题有点难。

---

# 比赛题目

## [A. 跳跃游戏](https://ac.nowcoder.com/acm/contest/56825/A)

大概的题意，从数组下标0开始，到末尾结束，只要存在一个严格递增序列，则输出YES，否则NO。

注意这边不限定跳跃步长。

总之看到这题，一眼DP，二眼懵逼。

如果不限定步长的话，那不就简单比较加，$arr[0] 和 arr[n - 1]$的大小关系吗？

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        System.out.println(arr[n - 1] > arr[0] ? "YES" : "NO");
    }
    
}
```

如果该题，限定步长，那就有意思了，你会怎么解呢？

---

## [B. 数数](https://ac.nowcoder.com/acm/contest/56825/B)

求[1, n]内因子个数为奇数的个数

初读题, 感觉有些绕, 看例子才看明白.

只有自身为平方数的数,其因子个数才为奇数.

所以$\le n$的个数为$\sqrt n$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int t = sc.nextInt();
            System.out.println((int)Math.sqrt(t));
        }
    }

}
```

注：在整数范围内，不需要考虑$sqrt$操作带来的整数不准问题.

---

## [C. 操作数组](https://ac.nowcoder.com/acm/contest/56825/C)

求两个数组arr1，arr2，通过操作arr1中的任选两个数，一增一减，使得最后$arr1==arr2$的最小操作数？

一增一减，只能改变局部，但是不影响全局，因此这里的大前提，就是$\sum arr1==\sum arr2$

剩下就是贪心，多的找少的匹配，全局来看最终结果为$res=\sum_{i=1}^{i=n}(arr1[i]-arr2[i])/2$

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long[] arr1 = new long[n];
        long[] arr2 = new long[n];

        for (int i = 0; i < n; i++) {
            arr1[i] = sc.nextLong();
        }
        for (int i = 0; i < n; i++) {
            arr2[i] = sc.nextLong();
        }

        long s1 = Arrays.stream(arr1).sum();
        long s2 = Arrays.stream(arr2).sum();
        if (s1 != s2) {
            System.out.println("-1");
        } else {
            long diff = 0;
            for (int i = 0; i < n; i++) {
                diff += Math.abs(arr1[i] - arr2[i]);
            }
            System.out.println(diff / 2);
        }
    }

}
```

---

## [D. 遗迹探险](https://ac.nowcoder.com/acm/contest/56825/D)

迷宫只能向右向下，求最优解？

是不是很熟悉，DP入门题，但是出题人可没那么“人美心善”

地图中，存在多个传送门，但是传送门最**多使用一次**。

> 那如何快速求解从任一点(x,y)到(n,m)地图的最优解

其实可以逆向DP求解

引入一个downs用于保存从(1,1)到(x,y)的最优解

再引入一个ups用于保存从(n,m)到(x,y)的逆向最优解

所以该题的最终解，可以归纳为如下：

$max(max_{(x,y)\in S} (downs[x_i][y_i] + ups[x_j][y_j]), downs[n][m])$

$S为传送门集合$

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    static class Solution {

        long[][] downs;
        long[][] ups;

        // 预处理
        public void handle(int h, int w, long[][] grids) {
            downs = new long[h + 1][w + 1];
            ups = new long[h + 2][w + 2];

            for (int i = 0; i < downs.length; i++) Arrays.fill(downs[i], Long.MIN_VALUE);
            for (int i = 0; i < ups.length; i++) Arrays.fill(ups[i], Long.MIN_VALUE);
            
            downs[1][0] = downs[0][1] = 0;
            ups[h + 1][w] = ups[h][w + 1] = 0;

            for (int i = 1; i <= h; i++) {
                for (int j = 1; j <= w; j++) {
                    downs[i][j] = Math.max(downs[i - 1][j], downs[i][j - 1]) + grids[i][j];
                }
            }

            for (int i = h; i >= 1; i--) {
                for (int j = w; j >= 1; j--) {
                    ups[i][j] = Math.max(ups[i + 1][j], ups[i][j + 1]) + grids[i][j];
                }
            }
        }

        public long solve(int[][] ps) {
            // 不用传送门
            long ans = downs[downs.length - 1][downs[0].length - 1];
            // 枚举传送门组合
            for (int i = 0; i < ps.length; i++) {
                int r1 = ps[i][0], c1 = ps[i][1];
                for (int j = 0; j < ps.length; j++) {
                    if (i == j) continue;
                    int r2 = ps[j][0], c2 = ps[j][1];
                    ans = Math.max(ans, downs[r1][c1] + ups[r2][c2]);
                }
            }
            return ans;
        }

    }
    
    public static void main(String[] args) {

        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int h = sc.nextInt(), w = sc.nextInt();
        long[][] grid = new long[h + 1][w + 1];
        for (int i = 1; i <= h; i++) {
            for (int j = 1; j <= w; j++) {
                grid[i][j] = sc.nextLong();
            }
        }

        Solution solution = new Solution();
        solution.handle(h, w, grid);

        int z = sc.nextInt();
        for (int i = 0; i < z; i++) {
            int k = sc.nextInt();
            int[][] ps = new int[k][2];
            for (int j = 0; j < k; j++) {
                ps[j][0] = sc.nextInt();
                ps[j][1] = sc.nextInt();
            }

            long res = solution.solve(ps);
            System.out.println(res);
        }

    }
    
}
```


---

## [E. 顶级厨师](https://ac.nowcoder.com/acm/contest/56825/E)

如果数据量少的话，是可以用多路归并的思路来求解第K大

但这个数据量级真的太大，$n*m=10^{12}$, 没法模拟。

那如何求解呢？

如果正向不行，能否换个思路求解呢？

> 比如对于某个值x，他可以在这个组合序列中排第几大？

那这不就是二分的思路吗？

不过这里面，check的核心逻辑中，需要再次二分。

因此这次就是二分套二分，很经典。

不过这边还有一个难点，就是如何处理bans的组合

事实上，由于求第K大的值，**bans的关系组合pair并不重要，重要的值**

```java
import java.io.*;
import java.util.*;

public class Main {

    static int lowerBound(List<Long> foods, long cur, long m) {
        int l = 0, r = foods.size() - 1;
        while (l <= r) {
            int mid = l + (r - l) / 2;
            if (cur * foods.get(mid) <= m) {
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }
        return r + 1;
    }

    public static boolean check(long[] arr, List<Long>  food, List<Long> bans, long x, long m) {
        long acc = -lowerBound(bans, 1, m);
        for (int i = 0; i < arr.length; i++) {
            acc += lowerBound(food, arr[i], m);
            if (acc >= x) return true;
        }
        return acc >= x;
    }

    public static long solve(long[] arr, List<Long> food, List<Long> bans, long x) {

        long ans = 0;
        long mz = Arrays.stream(arr).max().getAsLong();
        long l = 1, r = mz * food.get(food.size() - 1);
        while (l <= r) {
            long m = l + (r - l) / 2;
            if (check(arr, food, bans, x, m)) {
                r = m - 1;
            } else {
                l = m + 1;
            }
        }
        return l;

    }

    public static void main(String[] args) {
        AReader sc = new AReader();
        PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));

        int n = sc.nextInt(), m = sc.nextInt();
        int k = sc.nextInt();
        int q = sc.nextInt();


        long[] arr = new long[n];
        List<Long> foods = new ArrayList<>();

        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        for (int j = 0; j < m; j++) {
            long v = sc.nextInt();
            foods.add(v);
        }

        List<Long> bans = new ArrayList<>();
        for (int i = 0; i < k; i++) {
           int u = sc.nextInt() - 1;
           int v = sc.nextInt() - 1;
            bans.add(arr[u] * foods.get(v));
        }

        Collections.sort(foods);
        Collections.sort(bans);

        for (int i = 0; i < q; i++) {
            long x = sc.nextLong();
            out.println(solve(arr, foods, bans, x));
        }
        out.flush();

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

## [F. 排座位](https://ac.nowcoder.com/acm/contest/56825/F)

好像不会，叹息之墙

---

# 写在最后

在开始之前怎么能就去想输了怎么办。那家伙嘲笑了我们的梦想，就是我们的敌人，我们两个一起干掉她吧！

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230513/446702330_1683942757801/D2B5CA33BD970F64A6301FA75AE2EB22)




