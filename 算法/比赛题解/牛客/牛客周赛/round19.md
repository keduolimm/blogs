#牛客周赛 Round 19 解题报告 | 珂学家

#前言


# 整体评价

这场挺有意思的，尤其是T4， 其实很早之前也想过这个问题？如何智能的扫雷，感觉有点难。这题被逼得主动去求解这个扫雷问题，幸好只有4*4，可以暴力枚举。喜欢这种比赛。

## A.  小红的字符串大小写变换
Q: API题，把前k个字符大写，后n-k个字符小写

可以切分为2段，然后分别大写，小写化，然后拼接即可

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        int k = sc.nextInt();
        String s = sc.next();
        System.out.println(
                s.substring(0, k).toUpperCase() + s.substring(k).toLowerCase()
        );
    }

}
```
---

## B. 小红杀怪
Q: 有两种技能，一种单体伤害，一种群体伤害，求消灭2个不同血量的怪物，最多使用多少技能？

这题的数据范围有点小，其实可以枚举群体伤害的次数，然后求单体伤害的次数，从而获得最小值。

切入点就是

枚举全体伤害次数，然后求的单体伤害
```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int a = sc.nextInt(), b = sc.nextInt();
        int x = sc.nextInt(), y = sc.nextInt();

        int ans = Math.max((a + y - 1) / y, (b + y - 1) / y);
        for (int i = 0; i <= ans; i++) {
            int t1 = (Math.max(a - y * i, 0) + x - 1) / x;
            int t2 = (Math.max(b - y * i, 0) + x - 1) / x;
            ans = Math.min(ans, i + t1 + t2);
        }
        System.out.println(ans);
    }

}
```
---

## C. 小红的元素分裂
Q: 元素x可以分裂为1, x-1,  或者a, b(x=a*b), 求最小次数

可以预处理，枚举一个数的因子，最多, 而因子数均摊为

可以顺推，其实也可以记忆化搜索

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static int SZ = 10_0000;
    static Integer[] memo = new Integer[SZ + 1];

    static int dfs(int u) {
        if (u == 1) return 0;
        if (memo[u] != null) return memo[u];

        int res = u;
        for (int i = 2; i <= u / i; i++) {
            if (u % i == 0) {
                int r1 = dfs(i);
                int r2 = dfs(u / i);
                res = Math.min(res, r1 + r2 + 1);
            }
        }
        int r3 = dfs(u - 1) + 1;
        res = Math.min(res, r3);
        return memo[u] = res;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long acc = 0;
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            acc += dfs(arr[i]);
        }
        System.out.println(acc);
    }

}
```

---

## D. 小红的扫雷游戏
Q: 给一个4x4的扫雷地图，还有一些开的数字格子，求得确定性的雷和非雷，标注‘X’和‘O’?

一开始想着，是否可以搞一个方程组

如果确定为雷和非雷，这些是可解的，还有一些不能明确解的。

然后想想，这我也不会呀，重新审视了下数据范围，4*4，还有雷/非雷, 这个0-1特性，所以想到了全枚举状态，然后进行验证。

如果一个合法的状态，可以对相应的格子进行打标签。

如果一个格子，即被打雷，又被打非雷，那就是不确定，反之就是确定态。

在来分析下时间复杂度为, , 这是理论上复杂度，实际上因为游戏本身的限制，并没有这个高。

写出来，还是挺有意思的，这个题目。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    static int[][] dirs = new int[][] {
            {-1, -1}, {-1, 0}, {-1, 1}, {0, -1}, {0, 1}, {1, -1}, {1, 0}, {1, 1}
    };

    static boolean verify(char[][] chess, int s) {
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                char c = chess[i][j];
                if (c == '.') continue;
                int p = c - '0';
                int mine = 0;
                for (int t = 0; t < dirs.length; t++) {
                    int y0 = i + dirs[t][0];
                    int x0 = j + dirs[t][1];
                    if (y0 >= 0 && y0 < 4 && x0 >= 0 && x0 < 4) {
                        int ts = y0 * 4 + x0;
                        if ((s & (1 << ts)) != 0) {
                            mine++;
                        }
                    }
                }
                if (mine != p) return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[][] chess = new char[4][];

        int notMine = (1 << 16) - 1;
        for (int i = 0; i < 4; i++) {
            chess[i] = sc.next().toCharArray();
            for (int j = 0; j < 4; j++) {
                if (chess[i][j] != '.') {
                    notMine -= 1 << (i * 4 + j);
                }
            }
        }

        // 收藏可能解, 1(0b01)一定是非雷，2(0b10)一定雷，3(0b11)则不确定
        int[][] vis = new int[4][4];

        // 枚举所有的可能性, 因为只有2^16种可能
        int range = 1 << 16;
        for (int i = 0; i < range; i++) {
            if ((i | notMine) == notMine && verify(chess, i)) {
                for (int r = 0; r < 4; r++) {
                    for (int c = 0; c < 4; c++) {
                        if ((i & (1 << (4 * r + c))) == 0) {
                            vis[r][c] |= 1;
                        } else {
                            vis[r][c] |= 2;
                        }
                    }
                }
            }
        }

        // 地雷图还原
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                if (chess[i][j] == '.') {
                    if (vis[i][j] == 1) chess[i][j] = 'O';
                    else if (vis[i][j] == 2) chess[i][j] = 'X';
                }
            }
        }

        for (int i = 0; i < 4; i++) {
            System.out.println(new String(chess[i]));
        }
    }

}
```

---

# 写在最后
