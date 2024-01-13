

# 前言

---

# 整体评价

G题，做了2小时，幸好最后求解出来了。


---

## [6. 期望次数](https://www.lanqiao.cn/problems/9493/learning/?contest_id=152)

思路: 只记得概率得正推，期望得逆推

所以这题的解题思路是
$$找到期望的转换公式$$

对于任一个$E(x)$

$$E(x) = p_0/p * (E(x) + 1) + p_1/p * (E(x * 2) + 1) .. + p_n/p * (E(x * n) + 1)$$

而当$y=x*n>m$时，则$E(y) = 0$

记忆化搜索的方式，感觉更容易写

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.math.BigInteger;
import java.util.HashMap;
import java.util.Map;
import java.util.StringTokenizer;

public class Main {

    static long mod = 998244353l;

    static class Tx {
        long fz;
        long fm;
        public Tx(long fz, long fm) {
            this.fz = fz;
            this.fm = fm;
        }

        public Tx mul(long f, long m) {
            return new Tx(fz * f % mod, fm * m % mod);
        }

        public Tx add(Tx other) {
            long nfz = (fz * other.fm % mod + other.fz * fm % mod) % mod;
            long nfm = fm * other.fm % mod;
            return new Tx(nfz, nfm);
        }
    }

    static class Solution {
        int n;
        long[] arr;
        long acc;
        long m;
        public Solution(long[] arr, long acc, long m) {
            this.n = arr.length;
            this.arr = arr;
            this.acc = acc;
            this.m = m;
        }

        Map<Long, Tx> memo = new HashMap<>();

        // 记忆化索索/逆推
        public Tx solve(long x) {
            if (memo.containsKey(x)) return memo.get(x);

            long tmp = arr[0];
            Tx eof = new Tx(0, 1);
            Tx cur = new Tx(0, 1);
            for (int i = 1; i < n; i++) {
                if (x * (i + 1) >= m) {
                    // 这个是最重要的优化
                    eof = new Tx(acc - tmp, acc);
                    break;
                } else {
                    Tx cs = solve(x * (i + 1));
                    cs = cs.add(new Tx(1, 1)).mul(arr[i], acc);
                    cur = cur.add(cs);
                    tmp += arr[i];
                }
            }
            cur = cur.add(new Tx(arr[0], acc));
            cur = cur.add(eof);

            Tx last = cur.mul(acc, acc - arr[0]);
            memo.put(x, last);
            return last;
        }

    }

    public static void main(String[] args) {
        AReader sc = new AReader();

        int n = sc.nextInt(), m = sc.nextInt();

        long acc = 0;
        long[] arr = new long[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
            acc += arr[i];
        }

        Solution solution = new Solution(arr, acc, m);
        Tx res = solution.solve(1);

        long result = res.fz * BigInteger.valueOf(res.fm).modInverse(BigInteger.valueOf(mod)).longValue() % mod;
        System.out.println(result);

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
    }

}
```








