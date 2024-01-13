
# 第 3 场 小白入门赛 解题报告 | 珂学家 | 单调队列优化的DP + 三指针滑窗

---

# 前言

---

# 整体评价

T5, T6有点意思，这场小白入门场，好像没真正意义上的签到，整体感觉是这样。

---

## [A. 召唤神坤](https://www.lanqiao.cn/problems/12117/learning/?contest_id=160)

思路: 前后缀拆解

```c++ []
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

int main()
{
  // 请在此输入您的代码
  int n;
  cin >> n;
  vector<int> arr(n);
  for (int i = 0; i < n; i++) {
    cin >> arr[i];
  }
  vector<int> pre(n);
  vector<int> suf(n);

  pre[0] = -1;
  for (int i = 1; i < n; i++) {
    pre[i] = max(arr[i - 1], pre[i - 1]);
  }
  suf[n - 1] = -1;
  for (int i = n - 2; i >= 0; i--) {
    suf[i] = max(arr[i + 1], suf[i + 1]);
  }

  int res = 0;
  for (int i = 1; i < n - 1; i++) {
    res = max(res, (pre[i] + suf[i]) / arr[i]);
  }
  cout << res << endl;

  return 0;
}
```

---

## [B. 聪明的交换策略](https://www.lanqiao.cn/problems/12118/learning/?contest_id=160)

思路: 模拟+枚举

```c++ []
#include <bits/stdc++.h>
using namespace std;

int main()
{
  // 请在此输入您的代码
  int n;
  cin >> n;
  string s;
  cin >> s;

  // long long res = 1LL << 60;

  long long acc1 = 0, acc2 = 0;
  int t0 = 0, t1 = 0;
  for (int i = 0; i < n;i++) {
    char c = s[i];
    if (c == '0') {
      acc1 += (i - t0);
      t0++;
    } else {
      acc2 += (i - t1);
      t1++;
    }
  }
  cout << min(acc1, acc2) << endl;

  return 0;
}
```

---

## [C. 怪兽突击](https://www.lanqiao.cn/problems/12116/learning/?contest_id=160)

思路: 枚举

```c++ []
#include <bits/stdc++.h>
using namespace std;
int main()
{
  // 请在此输入您的代码

  int n, k;
  cin >> n >> k;
  vector<int> arr(n), brr(n);
  for (int i = 0; i < n; i++) {
    cin >> arr[i];
  }
  for (int i = 0; i < n; i++) {
    cin >> brr[i];
  }

  long long res = 1LL << 60;

  long long pre = 0;
  long long mv = arr[0] + brr[0];
  for (int i = 0; i < n; i++) {
    if (i + 1 > k) break;
    pre += arr[i];
    mv = min(mv, (long long)arr[i] + brr[i]);
    res = min(res, pre + (k - i - 1) * mv);
  }
  cout << res << endl;

  return 0;
}
```

---

## [D. 蓝桥快打](https://www.lanqiao.cn/problems/12110/learning/?contest_id=160)

思路: 二分

```c++ []
#include <bits/stdc++.h>
using namespace std;

using int64 = long long;

int main()
{

  // 请在此输入您的代码
  int t;
  cin >> t;
  while (t-- > 0) {
    int a, b, c;
    cin >> a >> b >> c;

    // 可以二分的
    int l = 1, r = b;
    while (l <= r) {
      int m = l + (r - l) / 2;

      // *) 
      int times = (b + m - 1) / m;
      if ((int64)(times - 1) * c < a) {
        r = m - 1;
      } else {
        l = m + 1;
      }   

    }

    cout << l << endl;

  }

  return 0;

}
```

---

## [E. 奇怪的段](https://www.lanqiao.cn/problems/12112/learning/?contest_id=160)

思路: 单调队列优化的DP

这题只需要维护最大值就行，不需要维护单调队列

其核心是如下的公式

$$dp[i][j] = \max_{t=0}^{t=j-1} dp[i - 1][t] + (pre[j + 1] - pre[t]) * w[i]$$

公式拆解后

$$dp[i][j] = \max_{t=0}^{t=j-1}(dp[i - 1][t] - pre[t]) * w[i]) + pre[j] * w[i]$$

这样这个递推的时间代价为$O(1)$，而不是$O(n)$

这样总的时间复杂度为$O(n * k)$

```java []

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();

        int n = sc.nextInt(), p = sc.nextInt();
        int[] arr = new int[n];
        long[] pre = new long[n + 1];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
            pre[i + 1] = pre[i] + arr[i];
        }
        int[] ws = new int[p];
        for (int i = 0; i < p; i++) {
            ws[i] = sc.nextInt();
        }

        // *)
        long inf = Long.MIN_VALUE / 10;
        long[][] dp = new long[p + 1][n];
        for (int i = 0; i <= p; i++) {
            Arrays.fill(dp[i], inf);
        }

        for (int i = 0; i < n; i++) {
            dp[1][i] = pre[i + 1] * ws[0];
        }

        // O(n)
        // dp[i - 1][j] - p * pre[j + 1] + p * pre[j]
        for (int i = 2; i <= p; i++) {
            long tmp = dp[i - 1][0] - ws[i - 1] * pre[1];
            for (int j = 1; j < n; j++) {
                dp[i][j] = tmp + ws[i - 1] * pre[j + 1];
                tmp = Math.max(tmp, dp[i - 1][j] - ws[i - 1] * pre[j + 1]);
            }
        }

        System.out.println(dp[p][n - 1]);

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

## [F. 小蓝的反击](https://www.lanqiao.cn/problems/12114/learning/?contest_id=160)

思路: 滑窗 + 三指针

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.StringTokenizer;

public class Main {

    static List<int[]> split(int v) {
        List<int[]> res = new ArrayList<>();
        for (int i = 2; i <= v / i; i++) {
            if (v % i == 0) {
                int cnt = 0;
                while (v % i == 0) {
                    v /= i;
                    cnt++;
                }
                res.add(new int[] {i, cnt});
            }
        }
        if (v > 1) {
            res.add(new int[] {v, 1});
        }
        return res;
    }

    // *)
    static boolean check(int[][] pre, int s, int e, List<int[]> xx) {
        for (int i = 0; i < xx.size(); i++) {
            int tn = xx.get(i)[1];
            if (pre[i][e + 1] - pre[i][s] < tn) return false;
        }
        return true;
    }

    public static void main(String[] args) {
        AReader sc = new AReader();

        int n = sc.nextInt();
        int a = sc.nextInt();
        int b = sc.nextInt();

        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }
        if (b == 1) {
            System.out.println(0);
            return;
        }

        List<int[]> facs1 = split(a);
        int m1 = facs1.size();
        List<int[]> facs2 = split(b);
        int m2 = facs2.size();

        // 三指针做法
//        int[][] brr1 = new int[m1][n];
        int[][] pre1 = new int[m1][n + 1];

//        int[][] brr2 = new int[m2][n];
        int[][] pre2 = new int[m2][n + 1];

        for (int i = 0; i < n; i++) {
            int v = arr[i];
            for (int j = 0; j < m2; j++) {
                int p = facs2.get(j)[0];
                int tmp = 0;
                while (v % p == 0) {
                    v /= p;
                    tmp++;
                }
//                brr2[j][i] = tmp;
                pre2[j][i + 1] = pre2[j][i] + tmp;
            }

            v = arr[i];
            for (int j = 0; j < m1; j++) {
                int p = facs1.get(j)[0];
                int tmp = 0;
                while (v % p == 0) {
                    v /= p;
                    tmp++;
                }
//                brr1[j][i] = tmp;
                pre1[j][i + 1] = pre1[j][i] + tmp;
            }
        }

        long res = 0;
        int k1 = 0, k2 = 0;
        for (int k3 = 0; k3 < n; k3++) {

            // 找到不满足的点为止
            while (k1 <= k3 && check(pre1, k1, k3, facs1)) {
                k1++;
            }

            //
            while (k2 <= k3 && check(pre2, k2, k3, facs2)) {
                k2++;
            }

//            res += Math.min(k1, k2);
            // 0 - k1 - 1
            // k2 -> n

            res += (k2 <= k1 - 1) ? (k1 - k2): 0;
        }

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

