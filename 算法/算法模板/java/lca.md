

# 倍增版本

```java []
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.List;

public class Lca {

    int n, m;
    int[] depth;
    int[][] f;

    List<Integer> []g;

    public void setUpLca() {
        // 倍增
        m = (int)(Math.log(n) / Math.log(2)) + 1;
        f = new int[n + 1][m + 1];
        depth = new int[n + 1];

        Deque<Integer> deq = new ArrayDeque<>();
        deq.offer(1);
        depth[1] = 1;

        while (!deq.isEmpty()) {
            int now = deq.poll();
            for (int v: g[now]) {
                if (depth[v]!=0) continue;
                depth[v] = depth[now] + 1;
                f[v][0] = now;
                for (int i = 1; i <= m; i++) {
                    f[v][i] = f[f[v][i - 1]][i - 1];
                }
                deq.offer(v);
            }
        }
    }

    public int lca(int x, int y) {
        if (depth[x] > depth[y]) {
            // swap
            int t = x;
            x = y;
            y = t;
        }

        // y像x对齐 （高度)
        for (int i = m; i >= 0; i--) {
            if (depth[f[y][i]] >= depth[x]) y = f[y][i];
        }

        if (x == y) return x;

        for (int i = m; i >= 0; i--) {
            if (f[y][i] != f[x][i]) {
                y = f[y][i];
                x = f[x][i];
            }
        }
        return f[x][0];
    }

}

```
