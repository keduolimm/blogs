
# 并查集

并查集的应用有很多

- 普通并查集
  - 一般是图中联通性问题，分组，加权

- 种类并查集
  - 这个很特别，比如敌人的敌人是朋友  

- 链式并查集
  - 一般应用于遍历优化


```java
static class Dsu {
    int n;
    int[] arr;
    public Dsu(int n) {
        this.n = n;
        // 0-index
        this.arr = new int[n];
        Arrays.fill(arr, -1);
    }
    public int find(int u) {
        if (arr[u] == -1) {
            return u;
        }
        return arr[u] = find(arr[u]);
    }
    public void merge(int u, int v) {
        int a = find(u);
        int b = find(v);
        if (a != b) {
            arr[a] = b;
        }
    }

    public boolean isSame(int u,int v) {
        return find(u) == find(v);
    }
}
```