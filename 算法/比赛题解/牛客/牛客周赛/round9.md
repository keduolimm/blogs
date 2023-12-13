# 牛客周赛 Round 9 解题报告 | 珂学家 | 平均数定律


---

# 前言

![alt](https://uploadfiles.nowcoder.com/images/20230827/446702330_1693141901732/D2B5CA33BD970F64A6301FA75AE2EB22)


---

# 整体评价

C题只能模拟，好像直接用贡献法不行，如果要搞个O(n)时间复杂度还是挺难的。D题挺有趣的，名义上的众数，本质还是平均数构造，这题不是n个众数，就是n-1个众数。而n-1个众数，如何最小化代价挺费思量。


---

## [A. 小美的外卖订单编号](https://ac.nowcoder.com/acm/contest/63869/A)

因为涉及取模，所以最好的方式，是index 0，而不是index 1

所以对x先左偏移1位，取模后，在右偏移回来

> 形象一点就是：(x-1) % mod + 1

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int kase = sc.nextInt();
        while (kase-- > 0) {
            int m = sc.nextInt(), x = sc.nextInt();
            int res = ((x - 1) % m + 1);
            System.out.println(res);
        }
    }

}
```

---

## [B. 小美的加法](https://ac.nowcoder.com/acm/contest/63869/B)

就是枚举x号的位置，预处理累计和。

> min(sum - (arr[i] + arr[i + 1]) + arr[i] * arr[i + 1]), 1<=i<n-1

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long[] arr = new long[n];

        long sum = 0;
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            sum += arr[i];
        }
       
        // 取sum，因为可以不使用魔法
        long ans = sum;
        for (int i = 0; i < n - 1; i++) {
            ans = Math.max(ans, sum - arr[i] - arr[i + 1] + arr[i] * arr[i + 1]);
        }
        System.out.println(ans);
    }
}

```
---

## [C. 小美的01串翻转](https://ac.nowcoder.com/acm/contest/63869/C)

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

## [D. 小美的数组操作](https://ac.nowcoder.com/acm/contest/63869/D)

这题其实有一个思维点。

就是众多最多是多少？

其实细细分析下来， 这个数组的众数，不是n就是n-1，n的情况就是累加和能整除n，n-1就是剩下的情况。

在这个前提下，需要思考下，n-1的众数该如何构造，其代价最小呢？

我一开始的思路，就是枚举那个"倒霉鬼", 然后求中位数，平均数？

中位数定理 在这里是失效的，因为它有个+1,-1限制

所以尝试平均数，那这边的平均数avg，有个小陷阱，就是尝试向上，向下取整2个值。

```java
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Map;
import java.util.Scanner;
import java.util.TreeMap;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long[] arr = new long[n];
        long sum = 0;
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            sum += arr[i];
        }

        if (sum % n == 0) {
            // 能构造n个众数的情况
            long avg = sum / n;
            long op = 0;
            for (int i = 0; i < n; i++) {
                op += Math.abs(arr[i] - avg);
            }
            System.out.println(op / 2);
        } else {

            TreeMap<Long, Integer> range = new TreeMap<>();
            Arrays.sort(arr);
            long[] prev = new long[n + 1];
            for (int i = 0; i < n; i++) {
                prev[i + 1] = prev[i] + arr[i];
                range.put(arr[i], i);
            }

            long ans = Long.MAX_VALUE;
            // 枚举那个倒霉鬼
            for (int i = 0; i < n; i++) {
                long sum1 = sum - arr[i];
                // 向下取整
                long avg = sum1 / (n - 1);
                {
                    // 找到边界点
                    Map.Entry<Long, Integer> ent = range.floorEntry(avg);
                    int idx = ent.getValue();
                    // 左侧小于等于
                    long r1 = avg * (idx + 1) - prev[idx + 1];
                    if (i <= idx) {
                        // 去掉那个倒霉鬼的影响
                        r1 -= (avg - arr[i]);
                    }
                    // 右侧大于
                    long r2 = (prev[n] - prev[idx + 1]) - avg * (n - idx - 1);
                    if (i > idx) {
                        // 去掉那个倒霉鬼的影响
                        r2 -= (arr[i] - avg);
                    }
                    // 代价为两者的最大值
                    ans = Math.min(ans, Math.max(r1, r2));
                }

                // 向上取整
                avg = avg + 1;
                {
                    Map.Entry<Long, Integer> ent = range.floorEntry(avg);
                    int idx = ent.getValue();
                    long r1 = avg * (idx + 1) - prev[idx + 1];
                    if (i <= idx) {
                        r1 -= (avg - arr[i]);
                    }
                    long r2 = (prev[n] - prev[idx + 1]) - avg * (n - idx - 1);
                    if (i > idx) {
                        r2 -= (arr[i] - avg);
                    }
                    ans = Math.min(ans, Math.max(r1, r2));
                }
            }
            System.out.println(ans);

        }

    }

}

```
这个时间复杂度为$O(nlog(n))$

但实际上，这里可以继续贪心，因为这个倒霉鬼，一定是最小值，或者最大值，他们承受了转移的代价。

就是排序后的掐头去尾，枚举平均数。

--- 

# 写在最后

![alt](https://uploadfiles.nowcoder.com/images/20230827/446702330_1693141839711/D2B5CA33BD970F64A6301FA75AE2EB22)


