

# 欧拉降幂

![](https://uploadfiles.nowcoder.com/images/20231021/446702330_1697844260077/D2B5CA33BD970F64A6301FA75AE2EB22)

```java
class Euler {

    static long gcd(long a, long b) {
        return b == 0 ? a : gcd(b, a % b);
    }

    static long euler(long a, long b, long c, long p) {
        long fp = phi(p);
        long g = gcd(a, p);
        if (g == 1) {
            return ksm(a, ksm(b, c, fp), p);
        } else {
            double e = (Math.log(fp) / Math.log(b));
            if (c < e) {
                return ksm(a, (long)Math.pow(b, c), p);
            } else {
                return ksm(a, ksm(b, c, fp) + fp, p);
            }
        }
    }

    static long euler(long a, long b, long p) {
        long fp = phi(p);
        long g = gcd(a, p);
        if (g == 1) {
            return ksm(a, b % fp, p);
        } else {
            if (b < fp) {
                return ksm(a, b, p);
            } else {
                return ksm(a, b % fp + fp, p);
            }
        }
    }

    static long phi(long n) {
        long r = n;
        for (int i = 2; i <= n / i; ++i) {
            if (n % i == 0) {
                r = r / i * (i - 1);
                while (n % i == 0) {
                    n /= i;
                }
            }
        }
        if (n > 1) {
            r = r / n * (n - 1);
        }
        return r;
    }

    static long ksm(long b, long v, long mod) {
        long r = 1;
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