
# 第 381 场周赛 解题报告 | 珂学家 | 贪心构造 + 分类讨论&差分技巧

---

# 前言


![image.png](https://pic.leetcode.cn/1705812917-vxPzyA-image.png)


---

# 整体评价

前三题还是蛮快的，T4不难，但是细节多，有点琐碎了。大致思路为分类讨论，构建出环和两条链条，然后两两组合，借助差分加速计算.


---

## [T1. 输入单词需要的最少按键次数 I](https://leetcode.cn/contest/weekly-contest-381/problems/minimum-number-of-pushes-to-type-word-i/)

和T3一起讲

---

## [T2. 按距离统计房屋对数目 I](https://leetcode.cn/contest/weekly-contest-381/problems/count-the-number-of-houses-at-a-certain-distance-i/)

思路: floyd

因为n=100，虽然图边非常的稀疏，但是时间复杂度可以接受

```java []
class Solution {
    public int[] countOfPairs(int n, int x, int y) {
        int inf = Integer.MAX_VALUE / 10;
        int[][] path = new int[n][n];
        for (int i = 0; i < n; i++) {
            Arrays.fill(path[i], inf);
            path[i][i] = 0;
        }
        for (int i = 0; i < n - 1; i++) {
            path[i][i + 1] = path[i + 1][i] = 1;
        }

        // 这个很重要
        if (x != y) {
            path[x - 1][y - 1] = path[y - 1][x - 1] = 1;
        }

        // floyd
        for (int k = 0; k < n; k++) {
            for (int i = 0; i < n; i++) {
                for (int j = 0; j < n ; j++) {
                    if (path[i][j] > path[i][k] + path[k][j]) {
                        path[i][j] = path[i][k] + path[k][j];
                    }
                }
            }
        }

        // 统计
        int[] res = new int[n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (i == j) continue;
                res[path[i][j] - 1]++;
            }
        }

        return res;
    }
}
```

---

## [T3. 输入单词需要的最少按键次数 II](https://leetcode.cn/contest/weekly-contest-381/problems/minimum-number-of-pushes-to-type-word-ii/)

思路: 贪心构造

第一眼以为是哈夫曼树，细细一看，好像不是

就是按频率高低，然后分配到最小的slot(总共8个)

```java []
class Solution {
    public int minimumPushes(String word) {        
        int[] hash = new int[26];
        for (char c: word.toCharArray()) {
            hash[c - 'a']++;
        }
        List<Integer> lists = new ArrayList<>();
        for (int i = 0; i < 26; i++) {
            if (hash[i] > 0) lists.add(hash[i]);
        }
        Collections.sort(lists, Comparator.comparing(x -> -x));
        
        int res = 0;
        for (int i = 0; i < lists.size(); i++) {
            res += (i / 8 + 1) * lists.get(i);
        }
        return res;
    }
}
```

---
## [T4. 按距离统计房屋对数目 II](https://leetcode.cn/contest/weekly-contest-381/problems/count-the-number-of-houses-at-a-certain-distance-ii/)

思路: 分类讨论 + 差分加速

可以提取3个联通结构

- 一个环
- 两条链

先各自计算各自的组合

然后计算两两之间的组合贡献

组合计算时，可以借助差分把$O(N^2)$优化为$O(N)$.

这边还要讨论环个数点的奇偶性

所以导致这题的分类讨论情况有点多.

```java []
class Solution {
    public long[] countOfPairs(int n, int x, int y) {
        // from 1-index to 0-index
        x--; y--;
        // swap
        if (x > y) {
            int t = x; x = y; y = t;
        }

        long[] res = new long[n];
        if (x == y) {
            // 特判
            for (int i = 0; i < n; i++) {
                res[i] = (n - 1 - i) * 2;
            }
        } else {
            // 找到环
            // 环中的个数
            int m = Math.abs(x - y) + 1;
            if (m % 2 == 0) {
                int e = m / 2;
                for (int i = 0; i < e - 1; i++) {
                    res[i] += (long)m * 2;
                }
                res[e - 1] += m;
            } else {
                int e = m / 2;
                for (int i = 0; i < e; i++) {
                    res[i] += (long)m * 2;
                }
            }

            // 对单链x的处理
            for (int i = 0; i < x - 1; i++) {
                res[i] += (long)(x - 1 - i) * 2;
            }
            // 对y进行对称转换
            // 对单链y的处理
            y = n - 1 - y;
            for (int i = 0; i < y - 1; i++) {
                res[i] += (long)(y - 1 - i) * 2;
            }

            long[] diff = new long[n + 1];
            // 左侧存在链，和环组合
            if (x > 0) {
                if (m % 2 == 0) {
                    for (int i = 0; i < x; i++) {
                        res[(x - i) - 1] += 2;
                        diff[(x - i)] += 4;
                        diff[(x - i) + m / 2 - 1] -= 4;
                        res[(x - i) + m / 2 - 1] += 2;
                    }
                } else {
                    for (int i = 0; i < x; i++) {
                        res[(x - i) - 1] += 2;
                        diff[(x - i)] += 4;
                        diff[(x - i) + m / 2] -= 4;
                    }
                }
            }
            // 右侧存在链，和环组合
            if (y > 0) {
                if (m % 2 == 0) {
                    for (int i = 0; i < y; i++) {
                        res[(y - i) - 1] += 2;
                        diff[(y - i)] += 4;
                        diff[(y - i) + m / 2 - 1] -= 4;
                        res[(y - i) + m / 2 - 1] += 2;
                    }
                } else {
                    for (int i = 0; i < y; i++) {
                        res[(y - i) - 1] += 2;
                        diff[(y - i)] += 4;
                        diff[(y - i) + m / 2] -= 4;
                    }
                }
            }

            // 左右侧链的组合
            for (int i = 0; i < x; i++) {
                diff[(x - i) + 1] += 2;
                diff[(x - i) + 1 + y] -= 2;
            }

            // 最后的差分统计
            long delta = 0;
            for (int i = 0; i < n; i++) {
                delta += diff[i];
                res[i] += delta;
            }
        }

        return res;
    }
}
```

---

# 写在最后



![image.png](https://pic.leetcode.cn/1705812776-TevPoI-image.png)

