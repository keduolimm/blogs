# 牛客小白月赛75 解题报告 | 珂学家 | 0-1BFS + 前缀和优化DP

---

# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230701/446702330_1688207919318/D2B5CA33BD970F64A6301FA75AE2EB22)

谁年少的时候没有轻狂过，我那时可是语出惊人呢。

---

# 整体评价

VP了这场比赛，挺有意思的，当然C题的DFS也比较典， D是0-1 BFS比较典， E是前缀和优化DP，当然也可以双指针来解决。

---

## [A. 上班](https://ac.nowcoder.com/acm/contest/60063/A)

签到题，可以换种说法，可能更接地气些

就是珂朵莉MM上班，先坐地铁（耗时X分钟）, 到站后2选1（走路Y分钟，骑共享单车Z分钟）到公司。

求最短时间

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int x = sc.nextInt();
        int y = sc.nextInt();
        int z = sc.nextInt();
        System.out.println(x + Math.min(y, z));
    }
    
}
```

---

## [B. 崇拜](https://ac.nowcoder.com/acm/contest/60063/B)

这题可以直接贪心，而不需要排序刷选

就是先处理所有崇拜的情况，那这个值肯定累加最大。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int a = sc.nextInt();
        int b = sc.nextInt();
        int ans = 0;
        for (int i = 0; i < n; i++) {
            int v = sc.nextInt();
            if (v > b) {
                ans += 3;
            }
        }
        System.out.println(ans);
    }

}

```

---

## [C. 方豆子](https://ac.nowcoder.com/acm/contest/60063/C)

很经典的DFS练习题

就是引入偏移量，以及层次(offsetY, offsetX, layer)

寻找到递归子问题

当前(offsetY, offsetX, layer)

则其4个子问题为

$delta=3 * pow(2, layer - 1)$

- (offsetY, offsetX, layer - 1)
- (offsetY + delta, offsetX, layer - 1)
- (offsetY, offsetX + delta, layer - 1)
- (offsetY + delta, offsetX + delta, layer - 1)

构建一个二维字节数组，使用DFS递归填充即可，再最后打印

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static char[][] GOOD = new char[][] {
            "******".toCharArray(),
            "******".toCharArray(),
            "******".toCharArray(),
            "***...".toCharArray(),
            "***...".toCharArray(),
            "***...".toCharArray(),
    };

    static char[][] BAD = new char[][] {
            "......".toCharArray(),
            "......".toCharArray(),
            "......".toCharArray(),
            "...***".toCharArray(),
            "...***".toCharArray(),
            "...***".toCharArray(),
    };

    static void dfs(char[][] res, int offsetY, int offsetX, int layer, boolean good) {
        if (layer == 1) {
            char[][] template = good ? GOOD : BAD;
            for (int i = 0; i < template.length; i++) {
                for (int j = 0; j < template[i].length; j++) {
                    res[i + offsetY][j + offsetX] = template[i][j];
                }
            }
            return;
        }

        int dd = 3 * (int)Math.pow(2, layer - 1);

        dfs(res, offsetY, offsetX, layer - 1 , !good);
        dfs(res, offsetY, offsetX + dd, layer - 1 , !good);
        dfs(res, offsetY + dd, offsetX, layer - 1 , !good);
        dfs(res, offsetY + dd, offsetX + dd, layer - 1 , good);

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt();
        int ln = 3 * (int)Math.pow(2, n);

        char[][] res = new char[ln][ln];

        dfs(res, 0, 0, n, true);

        for(int i = 0; i < ln; i++) {
            System.out.println(new String(res[i]));
        }

    }

}
```

---

## [D. 矩阵](https://ac.nowcoder.com/acm/contest/60063/D)

引入状态(y,x,state), y,x是坐标，state=0,1表示是否翻转

也是经典的0-1BFS，当然这边有些特殊，可能更准确地说1-2BFS更好，而且需要分层BFS技巧。

那为啥这样做就是对的呢？

其实上(y,x,state)内部已经包含了一个最优解的路径

翻转是否会影响别的状态？
不会，因为影响别的状态，比如会导致回路，回路一定非最优解。

因此基于0-1 BFS的解，一定是最后的最优解。

顺便推荐一道Atcoder上的 0-1 BFS的题，[Stronger Takahashi](https://atcoder.jp/contests/abc213/tasks/abc213_e)


```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

public class Main {

    static int[][] dirs = new int[][] {
            {-1, 0}, {1, 0}, {0, -1}, {0, 1}
    };

    public static int solve(char[][] grid) {
        int h = grid.length, w = grid[0].length;

        int inf = Integer.MAX_VALUE / 10;
        int[][][] opt = new int[h][w][2];
        for (int i = 0; i < h; i++) {
            for (int j = 0; j < w; j++) {
                opt[i][j][0] = opt[i][j][1] = inf;
            }
        }

        // 0-1 BFS 架构
        Deque<int[]> deq = new LinkedList<>();
        deq.offer(new int[] {0, 0, 0, 0});
        opt[0][0][0] = 0;

        int level = 0;
        while (!deq.isEmpty()) {

            List<int[]> next1 = new ArrayList<>();
            List<int[]> next2 = new ArrayList<>();

            while (!deq.isEmpty() && deq.peekFirst()[3] == level) {
                int[] cur = deq.pollFirst();

                int y = cur[0], x = cur[1];
                if (opt[y][x][cur[2]] < cur[3]) continue;

                char c = (cur[2] == 0) ? grid[y][x] : (char) ('0' + ('1' - grid[y][x]));

                for (int i = 0; i < dirs.length; i++) {
                    int y1 = y + dirs[i][0];
                    int x1 = x + dirs[i][1];
                    if (y1 >= 0 && y1 < h && x1 >= 0 && x1 < w) {
                        if (grid[y1][x1] != c) {
                            if (opt[y1][x1][0] > cur[3] + 1) {
                                opt[y1][x1][0] = cur[3] + 1;
                                next1.add(new int[]{y1, x1, 0, cur[3] + 1});
                            }
                        } else {
                            if (opt[y1][x1][1] > cur[3] + 2) {
                                opt[y1][x1][1] = cur[3] + 2;
                                next2.add(new int[]{y1, x1, 1, cur[3] + 2});
                            }
                        }
                    }
                }
            }

            for (int[] now: next1) {
                if (opt[now[0]][now[1]][now[2]] == now[3]) {
                    deq.addFirst(now);
                }
            }
            for (int[] now: next2) {
                if (opt[now[0]][now[1]][now[2]] == now[3]) {
                    deq.addLast(now);
                }
            }
            level++;
        }

        return Math.min(opt[h - 1][w - 1][0], opt[h - 1][w - 1][1]);
    }

    public static void main(String[] args) {
        AReader sc = new AReader();
        int h = sc.nextInt();
        int w = sc.nextInt();

        char[][] grid = new char[h][];
        for (int i = 0; i < h; i++) {
            grid[i] = sc.next().toCharArray();
        }

        System.out.println(solve(grid));
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

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }

}
```

---


## [E. 数数](https://ac.nowcoder.com/acm/contest/60063/E)

这题不能从直接构造数组的角度去计数

实际上构建数组arr和构建基于arr的前缀和数组brr，是等价关系

这是这个题非常重要的一个观察。

如果从前缀和数组brr出发，它的值发现可枚举

即$brr[i] = i * j, j\in[1,m]$

可以简单压缩下状态$j=j*i$，令opt[i][j]，其实是前i个元素，前缀和为j*i的总方案数

那状态如何转移呢？

$opt[i+1][s]=\sum opt[i][t]$

这边的t和s满足，$t*i+1<=s*(i+1), t*i+m<=s*(i+1)$

此时的上下界为

${t的上界lower=\lceil (s*(i+1) - 1) / i \rceil}$

${t的下界high=\lfloor (s*(i+1) - m) / i \rfloor}$

如果按照
$opt[i+1][s]=\sum_{t=lower}^{t=high} opt[i][t]$

那时间复杂度为$O(nm^2)$，不优化，妥妥TLE

上下边界内，所以项都满足(区间和)，因此可以用常见的DP优化技巧，前缀和来搞定

当然这边也可以使用双指针，滑动，两者是等价的。

最终的时间复杂度为$O(nm)$

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int m = sc.nextInt();

        long mod = 10_0000_0007l;
        // 前缀和
        // 1, 2
        long[][] opt = new long[2][m + 1];
        for (int i = 1; i <= m; i++) opt[0][i] = 1;

        int prev = 0, next = 1;
        for (int i = 1; i < n; i++) {
            Arrays.fill(opt[next], 0);

            long[] pre = new long[m + 2];
            for (int j = 0; j < m + 1; j++) {
                pre[j + 1] = (pre[j] + opt[prev][j]) % mod;
            }

            // 前缀和的预处理
            for (int j = 1; j <= m; j++) {
                int t = j * (i + 1);

                int start = ((t - m) + (i - 1)) / i;
                int end = Math.min(m, (t - 1) / i);

                start = Math.min(start, m);
                start = Math.max(0, start);
                end = Math.min(end, m);

                opt[next][j] = ((pre[end + 1] - pre[start]) % mod + mod) % mod;
            }

            prev = 1 - prev;
            next = 1 - next;
        }

        long res = Arrays.stream(opt[prev]).sum() % mod;
        System.out.println(res);

    }

}
```

---

## [F. 打牌](https://ac.nowcoder.com/acm/contest/60063/F)


---

# 写在最后

大丘丘病了，二丘丘叫，三丘丘采药，四丘丘熬。

![alt](https://uploadfiles.nowcoder.com/images/20230701/446702330_1688207948480/D2B5CA33BD970F64A6301FA75AE2EB22)

