# 牛客周赛 Round 11 解题报告 | 珂学家 | 线性dp+大剪枝


---
# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230910/446702330_1694352740436/D2B5CA33BD970F64A6301FA75AE2EB22)

---

# 整体评价

T3和round 9的T3重复了，好意外。T4有点意思，比赛中一度不敢下手，然后试试骗分，发现过了。后来才知道，原来元素两两不等，那基本就退化为$O(n^2)$了。

---
## [A. 小美的外卖订单编号](https://ac.nowcoder.com/acm/contest/64593/A)

index 1 / index 0的问题

先减1，再加1

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int m = sc.nextInt();
            int x = sc.nextInt();

            int r = (x - 1) % m + 1;
            System.out.println(r);
        }
    }

}

```

---

## [B. 小美的数组操作2](https://ac.nowcoder.com/acm/contest/64593/B)

模拟就行，最后判定一下

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int n = sc.nextInt(), m = sc.nextInt();
            int[] arr = new int[n];
            for (int i = 0; i < n; i++) {
                arr[i] = sc.nextInt();
            }
            for (int i = 0;i  < m; i++) {
                int u = sc.nextInt() - 1;
                int v = sc.nextInt() - 1;
                arr[u]++;
                arr[v]--;
            }

            boolean f = true;
            int prev = Integer.MIN_VALUE;
            for (int i= 0; i < n; i++) {
                if (prev > arr[i]) f = false;
                prev = arr[i];
            }
            System.out.println(f ? "Yes" : "No");
        }
    }

}
```


---
## [C. 小美的01串翻转](https://ac.nowcoder.com/acm/contest/64593/C)

重题了，复制下上份题解

枚举起点，然后线性模拟, 累加0/1开头的最小值代价即可。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;
 
public class Main {
 
    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
         
        long ans = 0;
        for (int i = 0; i < str.length; i++) {
            int c0 = 0, c1 = 0;
            for (int j = i; j < str.length; j++) {
                if ((j - i) % 2 == 0) {
                   c0 += str[j] == '0' ? 0 : 1;
                   c1 += str[j] == '1' ? 0 : 1;
                } else {
                   c0 += str[j] == '1' ? 0 : 1;
                   c1 += str[j] == '0' ? 0 : 1;
                }
                // 取两者最小值
                ans += Math.min(c0, c1);
            }
        }
        System.out.println(ans);
 
    }
 
}
```
这个时间复杂度为$O(n^2)$

那这题是否可以从贡献的角度切入呢？

感觉有的难，因为子数组它是取代价最小的，每个点的贡献值好像是离散的，并不是一个范围区间。



---
## [D. 小美的元素删除](https://ac.nowcoder.com/acm/contest/64593/D)

这题的切入点

- 序列两两为倍数
- 元素互不相等

排序后，后一个元素都是前一个元素的倍数

因为最大数为10^9, 而最小倍数为2

那么序列的长度最多为31

```java
int x = n - k;
if (x >= 31) {
  System.out.println(0);
  return;
}
```
我们把删除k个元素，转化为挑选n-k个

那定义 dp[i][s], 以i元素为末尾元素，且前排累计挑选s个 的方案数

同时为数组建图，这样可以减少状态转移的枚举数

这题的难点在于，如果没有元素彼此不相同的约束，那该如何解？

```java
import java.io.*;
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt(), k = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        int x = n - k;
        if (x >= 31) {
            System.out.println(0);
            return;
        }

        Arrays.sort(arr);
        long[][] opt = new long[n][x + 1];
        long mod = 10_0000_0007;

        // 建图
        List<Integer> []g = new List[n];
        Arrays.setAll(g, t -> new ArrayList<>());
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (arr[j] % arr[i] == 0) {
                    g[i].add(j);
                }
            }
        }

        // dp
        long ans = 0;
        for (int i = 0; i < n; i++) {
            opt[i][1] = 1;
            for (int j = 0; j < x; j++) {
                if (opt[i][j] == 0) continue; // 这个剪枝也很重要
                for (int v: g[i]) {
                    opt[v][j + 1] = (opt[v][j + 1] + opt[i][j]) % mod;
                }
            }
        }

        for (int i = 0; i < n;i++) {
            ans = (ans + opt[i][x]) % mod;
        }
        System.out.println(ans);

    }

}
```

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20230910/446702330_1694352764611/D2B5CA33BD970F64A6301FA75AE2EB22)






