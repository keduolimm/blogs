# 牛客周赛 Round 21 解题报告 | 珂学家 | 堆栈的妙用

---
# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20231126/446702330_1701008405818/D2B5CA33BD970F64A6301FA75AE2EB22)

---
# 整体评价

从A题中的Baidu, 可以猜到这场有几道题来自于百度校招。

其实B题有点意思，如果把十字星的范围放大，那就可以成为一个hard题。

D题也挺意思的，大概有两种思路，一种是从左到右枚举右端点，增量累加，一种是贡献思路。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---
## [A. 小红的Baidu](https://ac.nowcoder.com/acm/contest/70303/A)

字符串s，重排后，可以等价于“Baidu”

思路， 最小化表达式

```java
import java.io.*;
import java.util.*;
import java.util.stream.Collectors;

public class Main {

    // Java 对 char数组，比较特殊
    static String expression(String s) {
        return s.chars().sorted().mapToObj(x -> "" + ((char)x)).reduce("", (a, b) -> a + b);
    }

    public static void main(String[] args) {
        // Baidu的最小表达式
        String p1 = expression("Baidu");
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int t = sc.nextInt();
        while (t-- > 0) {
            String s = sc.next();
            String p2 = expression(s);
            if (p1.equals(p2)) {
                System.out.println("Yes");
            } else {
                System.out.println("No");
            }
        }
    }

}
```

---
## [B. 小红盖章](https://ac.nowcoder.com/acm/contest/70303/B)

因为十字星的影响范围小，所以可以正向模拟即可。

但是如果这题十字星的范围很大，那又该如何求解呢？

这题，可以用逆向思路，即后续操作的优先级最大，后续操作会覆盖前面的。

但是仅有逆向思维是不够的，还需要对范围遍历进行优化，这需要借助数据结构。

- 构建横向W个链式并查集，纵向H个链式并查集，根据逆序优先，把遍历的代价(4K)降到均摊O(1)
- treeset, 均摊代价为$O(log n)$

这边还是正向解法

```java
import java.io.*;
import java.util.*;

public class Main {

    static int[][] dirs = new int[][] {
            {-1, 0}, {-2, 0}, {1, 0}, {2, 0},
            {0, -1}, {0, -2}, {0, 1}, {0, 2}
    };

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int h = sc.nextInt(), w = sc.nextInt();
        int k = sc.nextInt();

        char[][] grid = new char[h][w];
        for (int i = 0; i < h; i++) {
            Arrays.fill(grid[i], '.');
        }

        // 正向操作
        for (int i = 0; i < k; i++) {
            int y = sc.nextInt() - 1, x = sc.nextInt() - 1;
            char g = sc.next().charAt(0);
            grid[y][x] = g;
            for (int j = 0; j < dirs.length; j++) {
                int y0 = y + dirs[j][0];
                int x0 = x + dirs[j][1];
                if (y0 >= 0 && y0 < h && x0 >= 0 && x0 < w) {
                    grid[y0][x0] = g;
                }
            }
        }

        for (int i = 0; i < h; i++) {
            System.out.println(new String(grid[i]));
        }
    }

}
```

---

### 逆序+链式并查集

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    // 链式并查集
    static class Dsu {
        int n;
        int[] arr;
        public Dsu(int n) {
            this.n = n + 1;
            this.arr = new int[n + 2];
            Arrays.fill(arr, -1);
        }
        int leader(int u) {
            if (arr[u] == -1) return u;
            return arr[u] = leader(arr[u]);
        }
        // 向右合并
        void merge(int u) {
           int ai = leader(u);
           int bi = leader(ai + 1);
           arr[ai] = bi;
        }
    }


    static class Tx {
        int x, y;
        char s;
        public Tx(int x, int y, char s) {
            this.x = x;
            this.y = y;
            this.s = s;
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int h = sc.nextInt(), w = sc.nextInt();
        int k = sc.nextInt();

        char[][] grid = new char[h][w];
        for (int i = 0; i < h; i++) {
            Arrays.fill(grid[i], '.');
        }

        Dsu[] rows = new Dsu[h];
        Arrays.setAll(rows, x -> new Dsu(w));
        Dsu[] cols = new Dsu[w];
        Arrays.setAll(cols, x -> new Dsu(h));

        Tx[] ops = new Tx[k];
        for (int i = 0; i < k; i++) {
            int x = sc.nextInt() - 1, y = sc.nextInt() - 1;
            char g = sc.next().charAt(0);
            ops[i] = new Tx(x, y, g);
        }

        int E = 2; // 十字的边长
        
        // 逆序
        for (int i = k - 1; i >= 0; i--) {
            Tx op = ops[i];
            // 水平方向
            Dsu row = rows[op.x];
            int s = Math.max(op.y - E, 0);
            int e = Math.min(op.y + E, w - 1);
            while (true) {
                int y0 = row.leader(s);
                if (y0 > e) break;
                if (grid[op.x][y0] == '.') {
                    grid[op.x][y0] = op.s;
                }
                row.merge(y0);
            }
            // 垂直方向
            Dsu col = cols[op.y];
            s = Math.max(op.x - E, 0);
            e = Math.min(op.x + E, h - 1);
            while (true) {
                int x0 = col.leader(s);
                if (x0 > e) break;
                if (grid[x0][op.y] == '.') {
                    grid[x0][op.y] = op.s;
                }
                col.merge(x0);
            }
        }

        for (int i = 0; i < h; i++) {
            System.out.println(new String(grid[i]));
        }
    }

}
```


---
## [C. 小红的数组修改](https://ac.nowcoder.com/acm/contest/70303/C)

枚举每个位子被修改，然后计算在限制内，可以修改几种方式，能满足需求。

这边有3个限制

- 被改变的数，必须小于等于p
- 必须改变，不能为原先的值
- 改变后，数组和为k的倍数

假设数组累加和S

则改变后

S - arr[i] + y = k * x

取模变换

(arr[i] - S) mod k = y mod k，  且0 < y <= p, y != arr[i]

求得，r = (arr[i] - S) mod k, r为非负整数

转化为 y = k * t + r, 此时 0 < y <= p, 求t的非整数解个数

因为y<=p, 因此个数为 基本盘为 (p - r) / k

但是这里面有两个边界

- 如果r>0, 则修改方案需要额外 +1
- 如果arr[i] % k == r, 则需要额外 -1

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int p = sc.nextInt();
        int x = sc.nextInt();

        long acc = 0;
        long[] arr = new long[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
            acc += arr[i];
        }

        // 枚举
        long res = 0;
        for (int i = 0; i < n; i++) {
            long left = (acc - arr[i]) % x;

            // r为对应的同余
            long r = (left == 0) ? 0 : (x - left);
            if (p >= r) {
                res += (p - r) / x;
                if (r > 0) res++; // 需要补上1个

                // 需要去除相等的情况
                if (arr[i] <= p && arr[i] % x == r) res--;
            }
        }
        System.out.println(res);
    }

}
```

---
## [D. 括号匹配问题](https://ac.nowcoder.com/acm/contest/70303/D)

大致有两种思路，但无一例外都借助stack来实现。

以往做括号相关的题时，往往会引入stack来实现，这次也是如此。

### 方法1：递推函数

S(i) 为 以i节点为右端点的所有子区间的（）配对数

则S(i+1) 和 S(i) 的递进关系如何维护？

- 如果str[i+1]为'('

那么 S(i+1)=S(i), 即配对数完全复制，但是没有新增加

- 如果str[i+1]为’)‘

那这个’)', 可与左侧多余的’（‘构建一个新的配对关系，那具体能贡献多少个新增呢？ 这又如何维护呢？

其实可以手玩一下，就能发现，在i处增加一个'(', 相当于stack push(i+1)个，而消耗匹配一个，相当于stack pop栈顶。

因此 S(i + 1) = S(i) + 2 * stack.pop()

而最终的结果为 $\sum S(i)$

```java
import java.io.BufferedInputStream;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        int n = str.length;

        long res = 0;
        long s = 0;
        Deque<Integer> stack = new ArrayDeque<>();

        // 可能要数据结构辅助
        for (int i = 0; i < n; i++) {
            char c = str[i];
            if (c == '(') {
                stack.push((i + 1));
            } else if (c == ')') {
                if (!stack.isEmpty()) {
                    int v = stack.pop();
                    s += 2l * v;
                }
            }
            res += s;
        }
        System.out.println(res);
    }

}
```
---

### 方法2：贡献法

看了下前排大佬，基本都是贡献法

就是找到一个新()匹配, 其可以在多少个区间中重复贡献。

也是基于stack

匹配的时候， $\sum stack.pop() * (n - i)$

```java
import java.io.BufferedInputStream;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        int n = str.length;

        long res = 0;
        Deque<Integer> stack = new ArrayDeque<>();

        // 可能要数据结构辅助
        for (int i = 0; i < n; i++) {
            char c = str[i];
            if (c == '(') {
                stack.push((i + 1));
            } else if (c == ')') {
                if (!stack.isEmpty()) {
                    int v = stack.pop();
                    res += 2l * v * (n - i);
;               }
            }
        }
        System.out.println(res);
    }

}

```


---

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20231126/446702330_1701008390174/D2B5CA33BD970F64A6301FA75AE2EB22)

