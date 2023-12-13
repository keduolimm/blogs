# 牛客周赛 Round 23 解题报告 | 珂学家 | 构造场 + 容斥/状态 0-1背包

---
# 前言

![alt](https://uploadfiles.nowcoder.com/images/20231211/446702330_1702227877082/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 题解

前三题都是构造类型的题，倒是D题是很典的动态规划题。

牛客还是偏思维，偏构造，偏数学，T_T.

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---
## [A. 小红的整数转换](https://ac.nowcoder.com/acm/contest/71593/A)

a次操作+b，且a，b都是正整数

等价于 a * b = y - x

分类讨论下

- y - x > 0

  a=1, b=y-x，是可行解

- y - x <= 0

  无解



```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int t = sc.nextInt();
        while(t-- > 0) {
            int x = sc.nextInt(), y = sc.nextInt();
            if (y - x > 0) {
                System.out.printf("%d %d\n", 1, (y - x));
            } else {
                System.out.println("-1 -1");
            }
        }
    }
}
```

---
## [B. 小红打O](https://ac.nowcoder.com/acm/contest/71593/B)

找规律，找到'*'和坐标x,y的映射关系

可以分成3部分，然后找找对称性，这样可以方便构造。

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        char[][] grid = new char[4 * n][4 * n];
        for (int i = 0; i < 4 * n; i++) Arrays.fill(grid[i], '.');

        // 头和尾巴
        for (int i = 0; i < n; i++) {
            for (int j = n - i; j < 2 * n; j++) {
                grid[i][j] = grid[i][4 * n - 1 - j] = '*';
                grid[4 * n - 1 - i][j] = grid[4 * n - 1 - i][4 * n - 1 - j] = '*';
            }
        }
        
        // 中间
        for (int i = n; i < 3 * n; i++) {
            for (int j = 0; j < n; j++) {
                grid[i][j] = grid[i][4 * n - 1 - j] = '*';
            }
        }

        for (int i = 0; i < 4 * n; i++) {
            System.out.println(new String(grid[i]));
        }
    }

}

```


---
## [C. 小红的完全二叉树构造](https://ac.nowcoder.com/acm/contest/71593/C)

按照定义，父子之间必然有一个是偶数

对于给定的n，奇数一定不小于偶数的个数

所以，一种很自然的想法是

> 按树的奇偶分层，做二分图构建

根据奇数层的节点数，和偶数层的节点数的大小，来决策偶数放树的那一层。

- 奇数层数不小于偶数层的节点数

则把偶数层优先放置偶数，其他则放剩下的

顺序不重要，奇偶是唯一关注的点。

```java
import java.io.*;
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();

        int[] depth = new int[n + 1];
        int n1 = 0, n2 = 0;

        for (int i = 1; i <= n; i++) {
            depth[i] = (depth[i / 2] + 1) %  2;
            if (depth[i] == 1) n1++;
            else n2++;
        }

        Deque<Integer> arr1 = new ArrayDeque<>();
        Deque<Integer> arr2 = new ArrayDeque<>();
        for (int i = 1; i <= n; i++) {
            if (i % 2 == 1) arr1.add(i);
            else arr2.add(i);
        }

        int[] res = new int[n + 1];
        if (n1 >= n2) {
            // 偶数层先放偶数
            for (int i = 1; i <= n; i++) {
                if (depth[i] == 0) {
                    res[i] = !arr2.isEmpty() ? arr2.pop() : arr1.pop();
                }
            }
            // 奇数层再放剩下的
            for (int i = 1; i <= n; i++) {
                if (depth[i] == 1) {
                    res[i] = !arr2.isEmpty() ? arr2.pop() : arr1.pop();
                }
            }
        } else {
            // 奇数层先放偶数
            for (int i = 1; i <= n; i++) {
                if (depth[i] == 1) {
                    res[i] = !arr2.isEmpty() ? arr2.pop() : arr1.pop();
                }
            }
            // 偶数层再放剩下的
            for (int i = 1; i <= n; i++) {
                if (depth[i] == 0) {
                    res[i] = !arr2.isEmpty() ? arr2.pop() : arr1.pop();
                }
            }
        }

        System.out.println(
                IntStream.range(1, n + 1)
                    .mapToObj(x -> String.valueOf(res[x]))
                    .collect(Collectors.joining(" "))
        );
    }

}
```

---
## [D. 小红的红蓝硬币](https://ac.nowcoder.com/acm/contest/71593/D)

这题有两个思路
- 状态 + 0-1背包
- 容斥 + 0-1背包

两者皆以0-1背包为基础，区别在于，一个正向求解, 一个是正难则反。

---

### 1. 状态 + 0-1背包

正向求解，引入3个状态

- 0, 只包含红色的币
- 1，只包含蓝色的币
- 2，同时包含了红色和蓝色

把状态迁移整合进0-1背包的迭代求解过程中

令 opt[i][j][s], 前i项金币中，当前总价值为j，状态为s的方案数

第一维，可以借助滚动数组来优化，这样空间复杂度为$O(3*k)$

时间复杂度为$O(3*k*n)$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), p = sc.nextInt();
        char[] str = sc.next().toCharArray();

        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        long mod = (long)1e9 + 7;
        long[][] dp = new long[3][p + 1];

        for (int i = 0; i < n; i++) {
            // 这是不选择带来的转移
            long[][] dp2 =new long[][] {
                    dp[0].clone(), dp[1].clone(), dp[2].clone()
            };
            char c = str[i];
            int v = arr[i];
            // 选择带来的转移
            if (c == 'B') {
                for (int j = 0; j <= p - v; j++) {
                    dp2[0][j + v] = (dp2[0][j + v] + dp[0][j]) % mod;
                }
                for (int j = 0; j <= p - v; j++) {
                    dp2[2][j + v] = (dp2[2][j + v] + dp[1][j] + dp[2][j]) % mod;
                }
                // 从空集中转移而来
                dp2[0][v] = (dp2[0][v] + 1) % mod;
            } else {
                for (int j = 0; j <= p - v; j++) {
                    dp2[1][j + v] = (dp2[1][j + v] + dp[1][j]) % mod;
                }
                for (int j = 0; j <= p - v; j++) {
                    dp2[2][j + v] = (dp2[2][j + v] + dp[0][j] + dp[2][j]) % mod;
                }
                // 从空集中转移而来
                dp2[1][v] = (dp2[1][v] + 1) % mod;
            }
            dp = dp2;
        }

        System.out.println(dp[2][p]);

    }

}
```

---

### 2. 容斥 + 0-1背包

红色/蓝色金币至少都有一个的方案数

> 总的方案 - 只含红色 - 只含蓝色

而3部分的独立计算，就是简单的0-1背包

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), p = sc.nextInt();
        char[] str = sc.next().toCharArray();

        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        long mod = (long)1e9 + 7;

        long[] dp1 = new long[p + 1];
        long[] dp2 = new long[p + 1];
        long[] dp3 = new long[p + 1];

        // 只有红色的金币
        dp1[0] = 1;
        for (int i = 0; i < n; i++) {
            int v = arr[i];
            if (str[i] == 'R') {
                for (int j = p - v; j >= 0; j--) {
                    dp1[j + v] = (dp1[j + v] + dp1[j]) % mod;
                }
            }
        }

        // 只有蓝色的金币
        dp2[0] = 1;
        for (int i = 0; i < n; i++) {
            int v = arr[i];
            if (str[i] == 'B') {
                for (int j = p - v; j >= 0; j--) {
                    dp2[j + v] = (dp2[j + v] + dp2[j]) % mod;
                }
            }
        }

        // 所有金币的方案数
        dp3[0] = 1;
        for (int i = 0; i < n; i++) {
            int v = arr[i];
            for (int j = p - v; j >= 0; j--) {
                dp3[j + v] = (dp3[j + v] + dp3[j]) % mod;
            }
        }

        // 容斥
        long res = ((dp3[p] - dp2[p] - dp1[p]) % mod + mod) % mod;
        System.out.println(res);
    }

}
```



---
# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20231211/446702330_1702227893317/D2B5CA33BD970F64A6301FA75AE2EB22)


