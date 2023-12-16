
# 矩阵幂

```java
class Matrix {
    long[][] arr;
    int r, c;
    Matrix(long[][] arr) {
        this.arr = arr;
        this.r = arr.length;
        this.c = arr[0].length;
    }

    Matrix mul(Matrix other, long mod) {
        int nr = this.r, nc = other.c;
        long[][] res = new long[nr][nc];
        for (int i = 0; i < nr; i++) {
            for (int j = 0; j < nc; j++) {
                long temp = 0;
                for (int k = 0; k < c; k++) {
                    temp = (temp + arr[i][k] * other.arr[k][j] % mod) % mod;
                }
                res[i][j] = temp;
            }
        }
        return new Matrix(res);
    }

    // 生成 E 矩阵
    static Matrix E(int n) {
        long[][] arr = new long[n][n];
        for (int i = 0; i < n; i++) {
            arr[i][i] = 1;
        }
        return new Matrix(arr);
    }

    static Matrix quickPow(Matrix base, long k, long mod) {
        Matrix r = Matrix.E(base.r);
        while (k > 0) {
            if (k % 2 == 1) {
                r = r.mul(base, mod);
            }
            k /= 2;
            base = base.mul(base, mod);
        }
        return r;
    }
}
```