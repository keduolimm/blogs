


---

# 前言

---

# 整体评价

F题，其实已经找到切入点，但是公式没推导出来，T_T.

---

## [A. 新年快乐](https://www.lanqiao.cn/problems/10014/learning/?contest_id=157)

思路: 签到

```python []
import os
import sys

print("2024 AK ")
```

---

## [B. 蓝桥圣诞树](https://www.lanqiao.cn/problems/10015/learning/?contest_id=157)

思路: 建图后，dfs搜索

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.StringTokenizer;

public class Main {

    static boolean ans = true;

    static int dfs(int u, int fa, List<Integer>[] g, char[] str) {
        int r = 1;
        int t = 0;
        for (int v: g[u]) {
            if (v == fa) continue;
            int cr = dfs(v, u, g, str);
            if (str[u] == str[v]) {
                r += cr;
                // t = Math.max(cr, t);
            }
        }
        // r += t;
        if (r >= 3) ans = false;
        return r;
    }

    public static void main(String[] args) {
        AReader sc = new AReader();

        int t = sc.nextInt();
        while (t-- > 0) {
            int n = sc.nextInt();
            char[] str = sc.next().toCharArray();
            List<Integer>[]g = new List[n];
            Arrays.setAll(g, x->new ArrayList<>());

            for (int i = 0; i < n - 1; i++) {
                int u = sc.nextInt() - 1, v = sc.nextInt() - 1;
                g[u].add(v);
                g[v].add(u);
            }

            ans = true;
            dfs(0, -1, g, str);
            System.out.println(ans ? "YES" : "NO");
        }

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

## [C. 空间复杂度](https://www.lanqiao.cn/problems/10472/learning/?contest_id=157)

思路: 模拟

```java []
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int t = scanner.nextInt();
        scanner.nextLine(); // Consume the newline after reading t

        for (int i = 0; i < t; i++) {
            solve(scanner);
        }
    }

    private static void solve(Scanner scanner) {
        String[] ss = scanner.nextLine().split(" ");
        long x = Long.parseLong(ss[0]);
        String s = ss[1];
        int y = Integer.parseInt(ss[2]);

        if (s.equals("KB")) {
            x *= 1024;
        }
        if (s.equals("MB")) {
            x *= 1024 * 1024;
        }

        System.out.println(x / y);
    }
}
```


---

## [D. 开关](https://www.lanqiao.cn/problems/10009/learning/?contest_id=157)

思路: 分子分母分别维护，逆元

最好还是用类封装

```java []
// package lanqiao.season;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.math.BigInteger;
import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();

        long mod = 998244353l;
        int n = sc.nextInt();
        long[] a = new long[n];
        long[] b = new long[n];
        long[] c = new long[n];
        long[] d = new long[n];

        for (int i = 0; i < n; i++) {
            a[i] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            b[i] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            c[i] = sc.nextInt();
        }
        for (int i = 0; i < n; i++) {
            d[i] = sc.nextInt();
        }

        long afz = 0;
        long afm = 1;
        for (int i = 0; i < n; i++) {
            long z = (afz * b[i] % mod + a[i] * afm % mod) % mod;
            long m = afm * b[i] % mod;
            afz = z;
            afm = m;
        }

        long bfz = 0;
        long bfm = 1;
        for (int i = 0; i < n; i++) {
            long z = (bfz * d[i] % mod + c[i] * bfm % mod) % mod;
            long m = bfm * d[i] % mod;
            bfz = z;
            bfm = m;
        }


        long rf = 0;
        long rm = 1l;

        {
            long z = rf * afm % mod + n * afz % mod * rm % mod;
            z = (z % mod + mod) % mod;
            long m = rm * afm % mod;

            rf = z;
            rm = m;
        }

        {
            long z = rf * bfm % mod + n * bfz % mod * rm % mod;
            z = (z % mod + mod) % mod;
            long m = rm * bfm % mod;

            rf = z;
            rm = m;
        }

        {
            long f1 = afz * bfz % mod;
            long z1 = afm * bfm % mod;

            long z = rf * z1 % mod - 2 * f1 * rm % mod;
            z = (z % mod + mod) % mod;

            long m = rm * z1 % mod;
            rf = z;
            rm = m;
        }
        System.out.println(rf * BigInteger.valueOf(rm).modInverse(BigInteger.valueOf(mod)).longValue() % mod);
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

## [E. 异或与求和](https://www.lanqiao.cn/problems/10010/learning/?contest_id=157)

思路: 拆位 + 前后缀递推

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

    public static void main(String[] args) {
        AReader sc = new AReader();

        int n = sc.nextInt();
        long[] arr = new long[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        long mod = 998244353l;
        long res = 0;
        long[] hash = new long[30];
        for (int i = 0; i < n; i++) {
            long cnt = (long)(n - 1 - i) * (n - 2 - i) / 2;
            cnt %= mod;
            long v = arr[i];
            for (int j = 0; j < 30; j++) {
                int t = (int)((v >> j) & 0x01);
                if (t == 0) {
                    res += (long)hash[j] * (1 << j) % mod * cnt % mod;
                    res %= mod;
                } else {
                    res += (long)(i - hash[j]) * (1 << j) % mod * cnt % mod;
                    res %= mod;
                }
                if (t == 1) hash[j]++;
            }
        }

        long[] hash2 = new long[30];
        for (int i = n - 1; i >= 0; i--) {
            long cnt = (long)(i) * (i - 1) / 2;
            cnt %= mod;
            long v = arr[i];
            for (int j = 0; j < 30; j++) {
                int t = (int)((v >> j) & 0x01);
                if (t == 0) {
                    res += (long)hash2[j] * (1 << j) % mod * cnt % mod;
                    res %= mod;
                } else {
                    res += (long)(n - 1 - i - hash2[j]) * (1 << j) % mod * cnt % mod;
                    res %= mod;
                }
                if (t == 1) hash2[j]++;
            }
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

## [F. 数字求和](https://www.lanqiao.cn/problems/10011/learning/?contest_id=157)

思路: 枚举

TODO

---

## [G. 集合统计](https://www.lanqiao.cn/problems/10012/learning/?contest_id=157)

TODO

---

# 写在最后

