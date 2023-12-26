# 牛客周赛 Round 22 解题报告 | 珂学家 | 思维构造 + 最小生成树

---
# 前言

![alt](https://uploadfiles.nowcoder.com/images/20231203/446702330_1701608141804/D2B5CA33BD970F64A6301FA75AE2EB22)


---
# 整体评价

C题这个构造题挺好的，赛中把-1写成No, 直接整不会了，T_T.

D题是一道很裸的最小生成树题，只需要一个小小的逆向思维，把删除操作转换为构建过程。

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 小红的漂亮串](https://ac.nowcoder.com/acm/contest/70996/A)

数据规模较小，直接暴力匹配即可，当然也可以使用API。

```java []
import java.io.BufferedInputStream;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        String s = sc.next();

        int cnt = 0;
        int pos = 0;
        while (cnt < 2 && (pos = s.indexOf("red", pos)) != -1) {
            pos += 3;
            cnt++;
        }
        System.out.println(cnt >= 2 ? "Yes": "No");
    }

}
```

```python []
s = input()

p = 0
cnt = 0
# find 和 index的区别
while cnt < 2 and s.find("red", p) != -1:
    cnt += 1
    p = s.find("red", p) + 1
    
print ("Yes" if cnt >= 2 else "No")

```


---

## [B. 小红的偶子串](https://ac.nowcoder.com/acm/contest/70996/B)

好像滑窗贪心是错误的

引入dp[i + 1], 表示以i结尾，且是偶数位的最长子串(为了处理方便，漂移了一位)

- dp[i + 1] = dp[i - 1] + 2,  str[i] == str[i - 1]

- dp[i + 1] = 0, str[i] != str[i - 1]


```java []
import java.io.BufferedInputStream;
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {

        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        char[] str = sc.next().toCharArray();
        int n = str.length;

        int res = 0;
        int[] dp = new int[n + 1];
        for (int i = 1; i < n; i++) {
            if (str[i] == str[i - 1]) {
                dp[i + 1] = dp[i - 1] + 2;
            }
            res = Math.max(res, dp[i + 1]);
        }

        System.out.println(res);

    }

}
```

```python []
s = input()

n = len(s)
dp = [0] * (n + 1)

dp[0] = 0
for i in range(0, n - 1):
    if s[i] == s[i + 1]:
        dp[i + 2] = max(dp[i + 2], dp[i] + 2)

print (max(dp))
```

---
## [C. 小红的数组构造](https://ac.nowcoder.com/acm/contest/70996/C)

首先可以排除掉不可能的情况

- 1~n的累加和 > x
- n > k

然后想如何构造这个神奇的数组呢？

可以先安排，1~n填充，即arr[i] = i

那这样还剩余

left = x - (n+1)*n/2

那如何处理这个剩余的部分呢？

其实可以对n进行乘除

- d = left / n
- r = left % n

把d赋值给每个元素，然后r这个尾巴相当于给最后的r个元素额外+1

这样的构造数组，应该是理想上最满足要求的数组了。

---
题外话：

赛中也采用了，类似后悔堆的思路，即先填充1~n，然后多余的从大到小进行置换，可以过，但是没法证明。

```java
import java.io.BufferedInputStream;
import java.util.Scanner;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));
        int n = sc.nextInt();
        long k = sc.nextLong(), x = sc.nextLong();

        long s = ((long)n) * (n + 1) / 2;
        if (s > x || n > k) {
            System.out.println(-1);
        } else {
            long[] arr = new long[n + 1];
            for (int i = 1; i <= n; i++) {
                arr[i] = i;
            }

            x -= s;
            long d = x / n;
            long v = x % n;

            for (int i = 1; i <= n; i++) {
                arr[i] += d;
                if (n - i < v) {
                    arr[i]++;
                }
            }

            // 对数组进行最后的check，保证每个元素都小于等于k
            boolean ok = true;
            for (int i = 1;i <= n; i++) {
                if (arr[i] > k) ok = false;
            }
            
            if (!ok) System.out.println("-1");
            else System.out.println(IntStream.range(1, n + 1).mapToObj(t -> String.valueOf(arr[t])).collect(Collectors.joining(" ")));
        }
    }

}
```


---
## [D. 小红的图上删边](https://ac.nowcoder.com/acm/contest/70996/D)

逆向思维，把删除转换为重头构建树，那这样就变成求最小生成树的过程了。

这边采用 kruskal算法，借助并查集来实现最小生成树的构建。

这边需要额外计算边权，其实就是统计每个点的2,5的因子数

w(u, v) = min(cnt5(u) + cnt5(v), cnt2(u) + cnt2(v))

整体的时间复杂度为$O(mlogm)$, 主要在排序这块

```java
import java.io.BufferedInputStream;
import java.util.*;

public class Main {

    static class Dsu {
        int n;
        int[] arr;
        int[] gz;
        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
            this.gz = new int[n + 1];
            Arrays.fill(gz, 1);
        }

        int find(int u) {
            if (arr[u] == 0) return u;
            return arr[u] = find(arr[u]);
        }

        void union(int u, int v) {
            int ai = find(u), bi = find(v);
            if (ai != bi) {
                arr[ai] = bi;
                gz[bi] += gz[ai];
            }
        }

        int gSize(int u) {
            return gz[find(u)];
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(new BufferedInputStream(System.in));

        int n = sc.nextInt(), m = sc.nextInt();

        Dsu dsu = new Dsu(n);
        long[] ws = new long[n + 1];
        int[] cnt5 = new int[n + 1];
        int[] cnt2 = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            ws[i] = sc.nextLong();

            long v = ws[i];
            while (v % 5 == 0)  {
                cnt5[i]++;
                v /= 5;
            }
            while (v % 2 == 0)  {
                cnt2[i]++;
                v /= 2;
            }
        }

        long res = 0;
        List<int[]> edges = new ArrayList<>();
        for (int i = 0; i < m; i++) {
            int u = sc.nextInt(), v = sc.nextInt();
            int w = Math.min(cnt5[u] + cnt5[v], cnt2[u] + cnt2[v]);

            // 引入边权
            edges.add(new int[] {u, v, w});
            res += w;
        }

        // 按边权从小到大排序
        Collections.sort(edges, Comparator.comparing(x -> x[2]));
        long tmp = 0;
        for (int[] e: edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dsu.find(u) != dsu.find(v)) {
                dsu.union(u, v);
                tmp += w;
                if (dsu.gSize(1) == n) {
                    break;
                }
            }
        }

        // 删边收益 = 总的权值 - 最小生成树的权值
        System.out.println(res - tmp);
    }

}
```

```python []
import heapq

class Dsu(object):
    
    def __init__(self, n: int):
        self.n = n 
        self.arr = [0] * (n + 1) # index-1
        self.group = n
        
    def find(self, u: int) -> int:
        if self.arr[u] == 0:
            return u
        self.arr[u] = self.find(self.arr[u])
        return self.arr[u]
    
    def union(self, u :int, v: int):
        a = self.find(u)
        b = self.find(v)
        if a != b:
            self.arr[a] = b
            self.group -= 1
            
    def groupNum(self) -> int:
        return self.group
            
            
n, m = list(map(int, input().split()))
arr = list(map(int, input().split())) 

# 逆向思维
cnt2 = [0] * (n + 1)
cnt5 = [0] * (n + 1)

for i in range(len(arr)):
    v = arr[i]
    while v % 2 == 0:
        v /= 2
        cnt2[i + 1] += 1
    while v % 5 == 0:
        v /= 5
        cnt5[i + 1] += 1

res = 0
edges = []
for i in range(m):
    u, v = list(map(int, input().split()))
    edges.append((min(cnt2[u] + cnt2[v], cnt5[u] + cnt5[v]), u, v))
    res += min(cnt2[u] + cnt2[v], cnt5[u] + cnt5[v])

dsu = Dsu(n)
heapq.heapify(edges)

while dsu.groupNum() > 1 and len(edges):
    row = heapq.heappop(edges)
    if dsu.find(row[1]) != dsu.find(row[2]):
        dsu.union(row[1], row[2])
        res -= row[0]

print (res)
```

---
# 写在最后


![alt](https://uploadfiles.nowcoder.com/images/20231203/446702330_1701608121053/D2B5CA33BD970F64A6301FA75AE2EB22)


