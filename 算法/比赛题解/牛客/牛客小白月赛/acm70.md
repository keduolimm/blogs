
# 牛客小白月赛70 解题报告 | 珂学家 | 博弈SG函数 + 树上背包

---

# 前言
![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230616/446702330_1686887464881/D2B5CA33BD970F64A6301FA75AE2EB22)

我会永远呆在桐人身边所以请不要怕。

---

# 整体评价

前几题中规中矩，到是C题一度眼前一亮，如果C题按照现实游戏中的来，求全局最优解，那估计有非常的有意思了。E是一道经典的博弈SG函数，F题则是一道树形DP（背包形态）。

---

## [A. 小d和答案修改](https://ac.nowcoder.com/acm/contest/53366/A)

给你一个字符串，把小写改成大写，大写改成小写。

签到题，唯一的亮点是case中，有一个"NTR", 作者真可爱

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] buf = sc.next().toCharArray();
        for (int i = 0; i < buf.length; i++) {
            if (Character.isLowerCase(buf[i])) {
                buf[i] = Character.toUpperCase(buf[i]);
            } else {
                buf[i] = Character.toLowerCase(buf[i]);
            }
        }
        System.out.println(new String(buf));
    }

}
```

---

## [B. 小d和图片压缩](https://ac.nowcoder.com/acm/contest/53366/B)

就是把4个格子(2x2)压缩为一个格子（平均值)

这样就从${h\times w}$ 压缩为 ${h/2\times w/2}$的矩阵，题目保证h，w皆为偶数

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), m = sc.nextInt();

        int[][] ori = new int[n][m];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                ori[i][j] = sc.nextInt();
            }
        }

        int[][] grids = new int[n/2][m/2];
        for (int i = 0; i < n; i += 2) {
            for (int j = 0; j < m; j += 2) {
                grids[i/2][j/2] = (ori[i][j] + ori[i][j + 1] + ori[i + 1][j] + ori[i + 1][j + 1]) / 4;
            }
        }

        for (int i = 0; i < grids.length; i++) {
            String r = Arrays.stream(grids[i])
              .mapToObj(x -> String.valueOf(x)).collect(Collectors.joining(" "));
            System.out.println(r);
        }

    }

}

```

---

## [C. 小d和超级泡泡堂](https://ac.nowcoder.com/acm/contest/53366/C)

如果不是描述的背景，或者这题真的非常简单，实际也很简单

关键是这句话

> 当火焰蔓延到另一个地方将要继续蔓延时，蔓延方式等同于在该地方重新放置炸弹。

那思路就是找到G所在的连通区域，然后统计草地的个数，这个dfs/bfs/并查集都行

```
import java.io.BufferedInputStream;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Scanner;

public class Main {

    static int[][] dirs = new int[][] {
            {-1, 0}, {1, 0}, {0, -1}, {0, 1}
    };

    static int bfs(int y, int x, char[][] grids, boolean[][] used) {
        Deque<int[]> deq = new ArrayDeque<>();
        deq.offer(new int[] {y, x});
        used[y][x] = true;

        int ans = 0;

        while (!deq.isEmpty()) {
            int[] cur = deq.poll();
            if (grids[cur[0]][cur[1]] == '!') ans++;

            for (int i = 0; i < dirs.length; i++) {
                int y1 = cur[0] + dirs[i][0];
                int x1 = cur[1] + dirs[i][1];
                if (y1 >= 0 && y1 < grids.length && x1 >= 0 && x1 < grids[0].length) {
                    if (grids[y1][x1] != '#' && !used[y1][x1]) {
                        used[y1][x1] = true;
                        deq.offer(new int[] {y1, x1});
                    }
                }
            }
        }

        return ans;

    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int h = sc.nextInt(), w = sc.nextInt();
        char[][] grids = new char[h][];
        for (int i = 0; i < h; i++) {
            grids[i] = sc.next().toCharArray();
        }

        int ans = -1;
        boolean[][] used = new boolean[h][w];
        for (int i = 0; ans == -1 && i < h; i++) {
            for (int j = 0; j < w; j++) {
                if (grids[i][j] == '@' && !used[i][j]) {
                    ans = bfs(i, j, grids, used);
                    break;
                }
            }
        }

        System.out.println(ans);

    }

}
```

我一直想的问题是，如果按照实际游戏运行，目标是求放置最少炸弹个数，清除所有的草地？

如果是这样的话，这题还是蛮难的。

---

## [D. 小d和孤独的区间](https://ac.nowcoder.com/acm/contest/53366/D)

一个0,1构成的数组，求连续子区间[l,r]其内只包含一个1的个数

经典的三指针做法，感觉这类题，牛客很喜欢考察

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) arr[i] = sc.nextInt();

        long ans = 0l;
        int j = 0, k = 0;
        int acc1 = 0, acc2 = 0;
        for (int i = 0; i < n; i++) {
            // 找右侧的左边界
            while (j < i || (j < n && acc1 + arr[j] == 0)) {
                acc1 += arr[j];
                j++;
            }
            // 找右侧的右边界
            while (k < i || (k < n && acc2 + arr[k] <= 1)) {
                acc2 += arr[k];
                k++;
            }

            if (j < n) {
                ans += (k - j);
            }

            if (arr[i] == 1) {
                acc1--;
                acc2--;
            }
        }
        System.out.println(ans);
    }

}

```
---

## [E. 小d的博弈](https://ac.nowcoder.com/acm/contest/53366/E)

很经典的题，对于${H\times W}进行水平/垂直切割（不能相等），然后舍弃大的块，保留小的块，无非操作的人为失败方。

感觉可以两种做法，
- 采用SG函数来求解
- 传统的极大极小解法

但是这里的H，W范围太大了，导致第二条路行不通

那就试试SG函数

因为有h，w两个维度，实际上两者互相独立，因此最终结果为 $sg(h) \oplus sg(w)$

关键是SG函数如何定义了

定义下一个状态，因为不能相等，所以要奇偶讨论

${SG(s)=mex(SG(s_i))}$

可以观察到SG(s)是递增，相邻元素+0，+1

那${SG(s)=mex(SG(s_i))}$ 可以等价于
- s为奇数，${SG(s) = SG(s/2)+1}$
- s为偶数，${SG(s) = SG((s-1)/2)+1}$
- s=[0,1], SG(s) = 0

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    // 0,1,2,3,4,5,6,7,8,9,10
    // 0,0,0,1,1,1,1,2,2,2,
    public static int sg(int v) {
        if (v <= 2) return 0;
        if (v % 2 == 0) {
            return sg((v - 1) / 2) + 1;
        } else {
            return sg(v/2) + 1;
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt();
            int m = sc.nextInt();
            System.out.println((sg(n) ^ sg(m)) != 0 ? "Alice" : "Bob");
        }
    }

}

```

---

## [F. 小d和送外卖](https://ac.nowcoder.com/acm/contest/53366/F)

小d在树型结构的图中送外卖，从1点出发（没有外卖要送），而且其可以最多m(<=50)个点放弃配送，求最短的旅途？

非常乌托邦的题目，哈哈，难为作者了

这题是典型的树形DP

就是定义一个状态

${opt[u][k], u为节点编号，k为以该节点为根的子树，放弃了k个外卖点 的最短旅途}$

状态转移的话，非常的像分组有限背包的形态

即
2 : 0)+ k_u\, 满足 \sum k_i + k_u = k  }$

其中 nums[v]为以v为根节点的子树中外卖点的个数

${k_u取决于当前u点是否为外卖点，是则可取0/1，不是只能取0}$


大概就是这个思路

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

public class Main {

    static class Solution {

        static final long inf = Long.MAX_VALUE / 100;

        int m;
        Set<Integer> need = new HashSet<>();
        List<List<Integer>> g = new ArrayList<>();

        int[] nums;
        long[][] opt;

        void dfs(int u, int fa) {

            nums[u] = need.contains(u) ? 1 : 0;

            int sz = 0;
            List<Integer> cs = g.get(u);
            for (int v: cs) {
                if (v == fa) continue;
                dfs(v, u);
                sz++;

                nums[u] += nums[v];
            }


            opt[u][0] = 0;
            if (need.contains(u)) {
                opt[u][1] = 0;
            }


            if (sz > 0) {
                for (int v: cs) {
                    if (v == fa) continue;

                    long[] right = new long[m + 1];
                    Arrays.fill(right, inf);

                    // 分组背包模型
                    for (int i = m; i >= 0; i--) {
                        long tv = opt[u][i];
                        for (int j = (m - i); j >= 0; j--) {
                            right[i + j] = Math.min(right[i + j], tv + opt[v][j] + (nums[v] > j ? 2 : 0));
                        }
                    }

                    System.arraycopy(right, 0, opt[u], 0, m + 1);
                }
            } 

        }

        long solve(int n, int m, int[][] edges, int[] arr) {

            for (int i = 0; i <= n; i++) {
                g.add(new ArrayList<>());
            }
            for (int[] e: edges) {
                g.get(e[0]).add(e[1]);
                g.get(e[1]).add(e[0]);
            }
            for (int v: arr) need.add(v);

            opt = new long[n + 1][m + 1];
            for (int i = 0; i <= n; i++) Arrays.fill(opt[i], inf);

            this.m = m;
            this.nums = new int[n + 1];

            dfs(1, -1);

            long ans = Long.MAX_VALUE;

            for (int i = 0; i <= m; i++) {
                ans = Math.min(ans, opt[1][i]);
            }

            return ans;
        }

    }

    // *)
    public static void main(String[] args) {
        AReader sc = new AReader();
        int n = sc.nextInt(), m = sc.nextInt();
        int[][] edges = new int[n - 1][2];
        for (int i = 0; i < edges.length; i++) {
            edges[i][0] = sc.nextInt();
            edges[i][1] = sc.nextInt();
        }
        int k = sc.nextInt();
        int[] arr = new int[k];
        for (int i = 0; i < k; i++) {
            arr[i] = sc.nextInt();
        }

        Solution solution = new Solution();
        long res = solution.solve(n, m, edges, arr);
        System.out.println(res);

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

# 写在最后

我一直相信着的，不，我现在也相信的，今后也一样，你是我的英雄，无论何时都会来拯救我。

![alt](https://uploadfiles.nowcoder.com/images/20230616/446702330_1686887496058/D2B5CA33BD970F64A6301FA75AE2EB22)


