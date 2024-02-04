

利用费马小定律求逆元

```java
quick_mi(v, mod - 2, mod)
```

模板代码

```java []
static 
class Factorial {
    int n;
    long mod;
    long[] fac, inv;

    public Factorial(int n, long mod) {
        this.n = n;
        this.mod = mod;
        this.fac = new long[n + 1];
        this.inv = new long[n + 1];

        fac[0] = 1;
        for (int i = 1; i <= n; i++) {
            fac[i] = fac[i - 1] * i % mod;
        }
        inv[n] = ksm(fac[n], mod - 2);
        for (int i = n - 1; i >= 0; i--) {
            inv[i] = inv[i + 1] * (i + 1) % mod;
        }
    }

    public long fac(int m) {
        return fac[m];
    }

    public long facInv(int m) {
        return inv[m];
    }

    public long inv(int m) {
        // 0没有逆元
        return inv[m] * fac[m - 1] % mod;
    }

    public long comb(int m, int k) {
        return fac[m] * inv[k] % mod * inv[m - k] % mod;
    }

    private long ksm(long b, long v) {
        long r = 1l;
        while (v > 0) {
            if (v % 2 == 1) {
                r = r * b % mod;
            }
            b = b * b % mod;
            v /= 2;
        }
        return r;
    }
}
```