# 牛客周赛 Round 3 解题报告 | 珂学家 | 贪心思维场

---
# 前言

![alt](https://uploadfiles.nowcoder.com/compress/mw1000/images/20230716/446702330_1689510051999/D2B5CA33BD970F64A6301FA75AE2EB22)

寒之不寒无水也，热之不热无火也。

---
# 整体评价

感觉比较简单，更加侧重于思维吧。和前几场的Round系列，风格不太一样。

---

## [A. 游游的7的倍数](https://ac.nowcoder.com/acm/contest/61570/A)

因为连续7个数，比如有一个数是7的倍数

因此从个位数中着手添加，是最好的选择.

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        String s = sc.next();

        // 从个位数着手, 应该更快, 连续7个数，比如有一个数是7的倍数
        for (int i = 0; i < 10; i++) {
            String s1 = s + (char)(i + '0');
            if (Long.valueOf(s1) % 7 == 0) {
                System.out.println(s1);
                break;
            }
        }

    }

}
```

---

## [B. 游游的字母串](https://ac.nowcoder.com/acm/contest/61570/B)

这题还是枚举，就枚举最后的结果字母，这样有26种情况

然后遍历每个字符，取其左侧/右侧移动的最小代价 总和

这样的时间复杂度为$O(26*n)$, 当然这题可以做到$O(n)$

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        String s = sc.next();

        long ans = Long.MAX_VALUE;
        for (int i = 0; i < 26; i++) {
            long tmp = 0;
            for (char c: s.toCharArray()) {
                int p = c - 'a';
                // 取左侧和右侧最小的偏移量
                tmp += Math.min(Math.abs(p - i), 26 - Math.abs(p - i));
            }
            ans = Math.min(ans, tmp);
        }
        System.out.println(ans);

    }

}

```

---

## [C. 游游的水果大礼包](https://ac.nowcoder.com/acm/contest/61570/C)

因为数据范围比较小，$n,m\lt 10^6$

所以这题，实际上可以枚举 水果礼包1的数量，然后求得当前情况下的最优价值

或者说，对于多变量的最优解思路，往往是固定一个变量（枚举），然后求在一个变量情况下的最优解

如果范围放大

$x + 2 * y \le n$

$2 * x + y \le m$

求 $a * x + b * y$ 最大

总得感觉这个函数是个凸函数，可以用三分搞，总之是种很奇怪的感觉

```java
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt(), m = sc.nextInt();
        int a = sc.nextInt(), b = sc.nextInt();

        long ans = 0;
        for (int i = 0; i <= m; i++) {
            if (n < i * 2) break;

            long bag1 = (long)i * a;
            int left = Math.min((n - i * 2), (m - i)/2);

            long bag2 = (long)left * b;

            ans = Math.max(ans, bag1 + bag2);
        }

        System.out.println(ans);

    }

}
```

---

## [D. 游游的矩阵权值](https://ac.nowcoder.com/acm/contest/61570/D)

贡献法，可以观察得到

在中间$(n-2)\times(n-2)$区域，可贡献4次机会

在边上，则能贡献3次机会

在角上，只能贡献2次机会

因此，尽量把最大的$(n-2)\times(n-2)$的数放在中间区域

然后次大的放在边上，而角上永远是1,2,3,4这个组合

这边主要是易错，易错的原因是大数取模，而且部分和可能需要用到逆元

```java
import java.io.BufferedInputStream;
import java.math.BigInteger;
import java.util.Scanner;

public class Main {

    static final long mod = 10_0000_0007l;

    public static long tx(long t) {
        return (t % mod + mod) % mod;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        // 算贡献分
        long n = sc.nextLong();
        long m = n * n % mod;

        long cn = tx(n - 2) * tx(n - 2) % mod;
        long en = tx(n - 2) * 4 % mod;

        long inv2 = BigInteger.valueOf(2).modInverse(BigInteger.valueOf(mod)).longValue();

        // 烂肚皮(最大的(n-2)*(n-2)个数放中间)
        long r1 = tx(m + m - cn + 1) * cn % mod * inv2 % mod * 4 % mod;

        // 银边(剩下最大的4*(n-2)个数放边)
        long r2 = tx(m - cn + m - cn - en + 1) * en % mod * inv2 % mod * 3 % mod;

        // 金角(1,2,3,4)
        long r3 = 20l; // 固定值 (1+2+3+4) * 2

        System.out.println(tx(r1 + r2 + r3));
    }

}

```

---

# 写在最后

只需记得，她永远是那位“兼具智慧与美貌的八重神子大人”就好。

![alt](https://uploadfiles.nowcoder.com/images/20230716/446702330_1689510185020/D2B5CA33BD970F64A6301FA75AE2EB22)
