

# 第 4 场 小白入门赛 解题报告 | 珂学家 | 数学概率



---
## [F. 机器人](https://www.lanqiao.cn/problems/12422/learning/?contest_id=166)

思路: 数学

纯高中/初中数学

排序需要思维力

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.math.BigInteger;
import java.util.StringTokenizer;

public class Main {

    // 快速幂
    static long ksm(long b, long v, long mod) {
        long r = 1l;
        while (v > 0) {
            if (v % 2 == 1) {
                r = r * b % mod;
            }
            v /= 2;
            b = b * b % mod;
        }
        return r;
    }

    static long MOD = 998244353l;

    static long tx(long x) {
        return (x % MOD + MOD) % MOD;
    }

    static long inv(long x) {
        return BigInteger.valueOf(x).modInverse(BigInteger.valueOf(MOD)).longValue() % MOD;
    }

    public static void main(String[] args) {
        AReader sc = new AReader();
        int n = sc.nextInt();

        long psum = 0;
        int[] p = new int[n];
        for (int i = 0; i < n; i++) {
            p[i] = sc.nextInt();
            psum += p[i];
        }
        if (psum == 0) {
            System.out.println(0);
            long r = 0;
            for (int i = 0; i < n ;i++) {
                r = r + (long)(i + 1) * (i + 1) % MOD;
                r %= MOD;
            }
            System.out.println(r);
            return;
        }
        
        long mod = 998244353l;

        long fz = 1;
        long fm = 1;
        for (int i = 0; i < n; i++) {
            long tmp = ksm(2, p[i], mod);
            fm = fm * tmp % mod;
        }

        // 1 / (1 - pk)
        long pk = fz * inv(fm) % mod;
        long spk = inv(tx(1 - pk)) % mod;
        long[] res = new long[n];

        long accFz = 1;
        for (int i = 0; i < n; i++) {
            long k2 = ksm(2, p[i], mod);
            res[i] = accFz * tx(k2 - 1) % mod * inv(k2) % mod;
            res[i] = res[i] * spk % mod;
            accFz = accFz * inv(k2) % mod;
        }
        
        int ptr = 0;
        int[] hash = new int[n];
        for (int i = 0; i < n; i++) {
            if (p[i] != 0) hash[i] = ptr++;
        }
        for (int i = 0; i < n; i++) {
            if (p[i] == 0) hash[i] = ptr++;
        }
        
        long res1 = 0;
        long res2 = 0;
        for (int i = 0; i < n; i++) {
            res1 = res1 + (i + 1) * res[i] % mod;
            res1 %= mod;

            res2 = res2 + (long)(hash[i] + 1) * (i + 1) % mod;
            res2 %= mod;
        }
        System.out.println(res1);
        System.out.println(res2);
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
