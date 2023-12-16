# 第 360 场周赛 解题报告 | 珂学家 | 贪心 + 基环树森林



---

# 前言

![image.png](https://pic.leetcode.cn/1693149247-qbTueW-image.png)

魔术抓住了所有人的眼球，而唯独我与她相互对视，时间也为此停止

---

# 整体评价

挺难的一场比赛，被这个基环树森林击沉了，T3也挺有意思的，这类二进制思维题都挺难的。

---

## [T1. 距离原点最远的点](https://leetcode.cn/contest/weekly-contest-360/problems/furthest-point-from-origin/)

贪心模拟，就是’_‘要么替换为’L‘, 要么替换为’R’

然后取两者的最大值。

```java
class Solution {
    
    public int furthestDistanceFromOrigin(String moves) {
        int r1 = moves.replaceAll("_", "L").chars().map(x -> x == 'L' ? -1 : 1).sum();
        int r2 = moves.replaceAll("_", "R").chars().map(x -> x == 'L' ? -1 : 1).sum();
        return Math.max(Math.abs(r1), Math.abs(r2));
    }
    
}
```

---

## [T2. 找出美丽数组的最小和](https://leetcode.cn/contest/weekly-contest-360/problems/find-the-minimum-possible-sum-of-a-beautiful-array/)

上周原题，只是数据规模变大了

就是[1, target/2], [target, target + n - target / 2 - 1]这两个区间和

```java
class Solution {
    public long minimumPossibleSum(int n, int target) {
        long sum = 0;
        for (int i = 1; i <= Math.min(n, target / 2); i++) {
            sum += i;
        }
        for (int i = 0; i < n - Math.min(n, target / 2); i++) {
            sum += (target + i);
        }
        return sum;
    }
}
```

---

## [T3. 使子序列的和等于目标的最少操作次数](https://leetcode.cn/contest/weekly-contest-360/problems/minimum-operations-to-form-subsequence-with-target-sum/)

这题$\sum a_i \ge target$, 那一定有解，否则无解。

把target拆分为二进制形态，然后从这个角度切入。

如果有对应位的2幂次，贪心配对即可，如果不匹配，就需要思考如何构造。

这边的不匹配，可以理解为借位思路。

可以从左侧借位，这个属于形式的解，不需要真实操作

而从右侧借位，则需要真实的操作。

这也是为什么，这题从低到高遍历的原因。

时间复杂度为$O(n + 30 * 30)$, 感觉均摊后更倾向于$O(n+30)$

```java
class Solution {

    public int minOperations(List<Integer> nums, int target) {
    
        var idMap = new HashMap<Integer, Integer>();
        for (int i = 0; i <= 30; i++) {
            idMap.put(1 << i, i);
        }
        
        // 构建31的slots
        int[] slots = new int[32];
        for (int v : nums) {
            slots[idMap.get(v)]++;
        }
        
        int res = 0;
        for (int i = 0; i <= 30; i++) {
            int t = (target >> i) & 0x01;
            if (t == 0) {
                // 往高幂次转换
                slots[i + 1] += slots[i] / 2; 
            } else if (t == 1) {
                if (slots[i] > 0) {
                    slots[i]--;
                    // 往高幂次转换
                    slots[i + 1] += slots[i] / 2;
                } else {
                    // 借位
                    boolean f = false;
                    int idx = -1;
                    for (int j = i + 1; j <= 31; j++) {
                        if (slots[j] > 0) {
                            slots[j]--;
                            idx = j;
                            f = true;
                            break;
                        }
                    }
                    if (!f) return -1;
                    
                    // 从右侧借位
                    for (int j = i; j < idx; j++) {
                        slots[j]++;
                        res++;
                    }
                }
            }
        }

        return res;
    }
    
}
```

从高到底其实也可以

核心在于 前缀和 和 target的大小关系

- 前缀和 < target 就需要借位，需要真实的操作
- 前缀和 == target, 就构造完毕
- 前缀和 > target，贪心前缀中最大的数来匹配target的幂次位

这个应该就是 $O(n+30*30)$

---

## [T4. 在传球游戏中最大化函数值](https://leetcode.cn/contest/weekly-contest-360/problems/maximize-value-of-function-in-a-ball-passing-game/)

首先要理清这个图结构

![image.png](https://pic.leetcode.cn/1693151347-qzAMYS-image.png)


虽然是踢皮球游戏，但最终的组织形态如上图。

因为每个点都有出度，所以每个点最终都会陷入一个命中注定的环。

不知道把它称呼为 基环树森林 是否合适。

而最终的解为，从每个点出发计算k长度的路径和。

一般由两部分构造， 树状链 + 环内循环节

- 对于圆内的点，环内循环节
  预处理环内的前缀和，这部分相对简单
- 对于树状链出发的点，则需要计算 树上路径 + 环内切入点 + 循环节
  这部分相对繁琐，如果从起点开始很难利用记忆化，因为k太大了，所以可以反向建树，从树根(环的入口出)出发，进行dfs.

那回到一切的原点

> 如何提取环，如何构建环外的树

这就需要利用到 基环树 的构建过程

利用时间戳技巧

```python
vis = [0] * n
timestamp = 1
for i in range(0, n):
    if vis[i] > 0:
        continue
    # 当亲的时间戳，用于比较遇到的旧的点，还是新的点
    checkpoint = timestamp 
    j = i
    while vis[j] == 0:
        vis[j] = timestamp++
        j = parent[j]

    # 是这一轮的点(需要构建环)
    if vis[j] >= checkpoint: 
        # 构建环
        # 处理尾巴的树链
    # 遇到了前几轮的点
    else:
        # 这部分都输树链的一部分
```

大概就是这个流程，预处理获取了环，以及树链结构

对于获取的环，进行前缀和预处理

树链结构进行DFS

![image.png](https://pic.leetcode.cn/1693185264-RbWhgW-image.png)


```java
class Solution {

    static class Circle {
        List<Integer> arr;
        long[] prev;
        Map<Integer, Integer> hash = new HashMap<>();

        public Circle(List<Integer> arr) {
            this.arr = arr;
            int n = arr.size();
            prev = new long[n + 1];
            for (int i = 0; i < n; i++) {
                prev[i + 1] = prev[i] + arr.get(i);
                hash.put(arr.get(i), i);
            }
        }

        long compute(int r, long k) {
            int size = arr.size();
            int idx = (hash.get(r) + 1) % size;

            long div = k / size;
            int rim = (int)(k % size);

            long r2 = 0;
            if (idx + rim - 1 < size) {
                r2 = prev[idx + rim] - prev[idx];
            } else {
                r2 = prev[size] - prev[idx];
                r2 += prev[rim - (size - idx)];
            }
            return r2 + div * prev[size];
        }
    }

    List<Integer> []g;
    long gAns = Long.MIN_VALUE;
    long k;

    void dfs(int u, int depth, long acc, int r, List<Integer> arr,Circle circle) {
        if (depth >= k) {
            acc -= arr.get(depth - (int)k);
            gAns = Math.max(gAns, acc);
        } else {
            gAns = Math.max(gAns, acc + circle.compute(r, k - (depth + 1)));
        }

        for (int v: g[u]) {
            arr.add(v);
            dfs(v, depth + 1, acc + v, r, arr, circle);
            arr.remove(arr.size() - 1);
        }
    }

    public long getMaxFunctionValue(List<Integer> receiver, long k) {
        k++;
        this.k = k;

        int n = receiver.size();
        this.g = new List[n];
        Arrays.setAll(g, x -> new ArrayList<>());

        int[] vis = new int[n];
        int timestamp = 1;
        List<Circle> circles = new ArrayList<>();

        for (int i = 0; i < n; i++) {
            // 找到环
            if (vis[i] > 0) continue;
            int checkpoint = timestamp;

            int j = i;
            while (vis[j] == 0) {
                vis[j] = timestamp++;
                j = receiver.get(j);
            }

            // 说明找到环了
            if (vis[j] >= checkpoint) {
                // 处于环中
                List<Integer> arr = new ArrayList<>();
                int cup = j; //
                do {
                    arr.add(j);
                    j = receiver.get(j);
                } while (j != cup);

                // *) 提供环入口
                Circle circle = new Circle(arr);
                circles.add(circle);

                j = i;
                while (j != cup) {
                    // 逆向建树
                    g[receiver.get(j)].add(j);
                    j = receiver.get(j);
                }
            } else {
                // 逆向建树
                j = i;
                while (vis[j] >= checkpoint) {
                    g[receiver.get(j)].add(j);
                    j = receiver.get(j);
                }
            }
        }

        // 环内进行循环
        for (Circle circle: circles) {
            for (int v: circle.arr) {
                // 枚举圈内的点
                gAns = Math.max(gAns, circle.compute(v, k));

                // 枚举圈内的点外挂的树链
                List<Integer> lists = new ArrayList<>() {{add(v); }};
                dfs(v, 0, v, v, lists, circle);
            }
        }

        return gAns;
    }
}
```

这样的时间复杂度就是$O(n)$

---

补充倍增的代码

第一次遇到这样处理循环节的问题。

```java
class Solution {
    
    public long getMaxFunctionValue(List<Integer> receiver, long k) {
        
        int n = receiver.size();
        int[][] f = new int[n][40];
        long[][] sum = new long[n][40];
        
        for (int i = 0; i < n; i++) {
            f[i][0] = receiver.get(i);
            sum[i][0] = i;
        }
        
        for (int i = 1; i < 40; i++) {
            for (int j = 0; j < n; j++) {
                f[j][i] = f[f[j][i - 1]][i - 1];
                sum[j][i] = sum[j][i - 1] + sum[f[j][i - 1]][i - 1];
            }
        }
        
        // 需要k++
        k++;
        long ans = 0;
        for (int t = 0; t < n; t++) {
            long tmp = 0;
            int p = t;
            for (int i = 39; i >= 0; i--) {
                if ((k & (1l << i)) != 0) {
                    tmp += sum[p][i];
                    p = f[p][i];
                }
            }
            ans = Math.max(ans, tmp);
        }
        
        return ans;
        
    }
    
}
```


---

# 写在最后

踏上旅途，记住旅途的意义

![image.png](https://pic.leetcode.cn/1693149434-uAWOBP-image.png)
