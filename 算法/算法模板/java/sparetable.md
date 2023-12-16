

# SpareTable

支持不可变数组的RMQ查询

- gcd
- min
- max

```java
class SparesTable {

    int[][] tables;
    BiFunction<Integer, Integer, Integer> callback;

    public SparesTable(int[] arr, BiFunction<Integer, Integer, Integer> callback) {
        int n = arr.length;
        int m = (int)(Math.log(n) / Math.log(2) + 1);
        tables = new int[m][n];
        this.callback = callback;

        for (int i = 0; i < n; i++) {
            tables[0][i] = arr[i];
        }
        for (int i = 1; i < m; i++) {
            int half = 1 << (i - 1);
            for (int j = 0; j + half < n; j++) {
                tables[i][j] = callback.apply(tables[i - 1][j], tables[i - 1][j + half]);
            }
        }
    }

    // 闭闭区间
    int query(int l, int r) {
        int t = (int)(Math.log(r - l + 1) / Math.log(2));
        return callback.apply(tables[t][l], tables[t][r - (1 << t) + 1]);
    }

}
```