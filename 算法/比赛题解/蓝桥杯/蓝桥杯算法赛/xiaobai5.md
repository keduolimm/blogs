
# 第 5 场 小白入门赛 解题报告 | 珂学家 | 找规律 + 矩阵幂

---
# 前言

---
# 整体评价

可以参考

[官方题解](https://zhuanlan.zhihu.com/p/682031932)

---
## [A. 十二生肖](https://www.lanqiao.cn/problems/16582/learning/?contest_id=168)

思路: 模拟

```python3 []
import os
import sys

print ("5")
```


---
## [B. 欢迎参加福建省大学生程序设计竞赛](https://www.lanqiao.cn/problems/16577/learning/?contest_id=168)

思路: 元组的统计

```python3 []
import os
import sys
from collections import Counter

n = int(input())

cnt = Counter()
for _ in range(n):
  x, y = list(map(int, input().split()))
  cnt[(x, y)] += 1

print (len(cnt))
```

---
## [C. 匹配二元组的数量](https://www.lanqiao.cn/problems/16583/learning/?contest_id=168)

思路: 移项转移

ai/j = aj/i => ai * i = aj * j

令 f(i) = ai * i

然后组合计算

$$\sum v_i * (v_i - 1) / 2$$

```python3 []
import sys
from collections import Counter

n = int(input())
arr = list(map(int, input().split()))

cnt = Counter()
for i in range(n):
  cnt[(i + 1) * arr[i]] += 1

res = 0
for (k, v) in cnt.items():
  if v >= 2:
    res += v * (v - 1) // 2

print (res)
```

---
## [D. 元素交换](https://www.lanqiao.cn/problems/16822/learning/?contest_id=168)

思路: 贪心

```python3 []
import os
import sys

n = int(input())
arr = list(map(int, input().split()))

c1, c2 = 0, 0
for i in range(n * 2):
  if arr[i] == 0:
    if i % 2 == 1:
      c1 += 1
  elif arr[i] == 1:
    if i % 2 == 1:
      c2 += 1

print (min(c1, c2))
```

---
## [E. 下棋的贝贝](https://www.lanqiao.cn/problems/16578/learning/?contest_id=168)

思路: 找规律

```java []
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        long n = sc.nextLong();
        if (n == 1) {
            System.out.println(0);
        } else if (n == 2) {
            System.out.println(2);
        } else if (n == 3) {
            System.out.println(4);
        } else {
            // n - m * m 格子
            long m = (long)Math.sqrt(n);
            long r = 0;
            if (m >= 2) {
                long delta = (m - 2) * (m - 2) * 4;
                r += delta;
                r += (m - 2) * 4 * 3 + 8;
            }

            long e = m;
            long left = n - m * m;
            for (int i = 0; left > 0 && i < 4; i++) {
                if (left >= e) {
                    r += e * 3 - 2 + e;
                    left -= e;
                } else {
                    r += left * 3 - 2 + left;
                    left = 0;
                }
            }
            if (left > 0) {
                r += left * 4;
            }
            System.out.println(r);
        }


    }
}
```

---

## [F. 方程](https://www.lanqiao.cn/problems/16576/learning/?contest_id=168)

思路: 矩阵幂

公式:

$$f(n) = k * f(n - 1) - f(n - 2)$$

```java []
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int t = sc.nextInt();
        long mod = (long)1e9 + 7;
        while (t-- > 0) {
            long n = sc.nextLong(), k = sc.nextLong();

            if (n == 1) {
                System.out.println(k % mod);
            } else if (n == 2) {
                //
                long r = (((k % mod) * (k % mod) - 2) % mod + mod) % mod;
                System.out.println(r);
            } else {
                long f1 = k % mod;
                long f2 = (((k % mod) * (k % mod) - 2) % mod + mod) % mod;

                // 3 = 1,
                long[] r = MatrixPower.matrixPower(n - 2, k, mod);

                long res = ((r[0] * f2 % mod + r[1] * f1 % mod) % mod + mod) % mod;
                System.out.println(res);
            }


        }
    }


    static class MatrixPower {
        public static long[] matrixPower(long n, long k, long mod) {
            long[][] M = {
                    {k, -1},
                    {1, 0}
            };
            long[][] I = {
                    {1, 0},
                    {0, 1}
            };
            while (n > 0) {
                if(n%2 == 1) {
                    I = matrixMultiply(I, M, mod);
                }
                n /= 2;
                M = matrixMultiply(M, M, mod);  // 平方M
            }
            return  new long[] { I[0][0], I[0][1]};  // 返回左上角的元素作为斐波那契数列的第n个值
        }

        public static long[][] matrixMultiply(long[][] A, long[][] B, long mod) {
            long[][] C = new long[A.length][B[0].length];
            for (int i = 0; i < A.length; i++) {
                for (int j = 0; j < B[0].length; j++) {
                    for (int k = 0; k < B.length; k++) {
                        C[i][j] += A[i][k] * B[k][j] % mod;
                        C[i][j] %= mod;
                    }
                }
            }
            return C;
        }
    }

}
```

---

# 写在最后

