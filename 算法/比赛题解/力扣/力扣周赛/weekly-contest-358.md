# 第 358 场周赛 解题报告 | 珂学家 | 单调栈贡献&贪心 + 分块上下界


---

# 前言

![image.png](https://pic.leetcode.cn/1691899413-wRVesM-image.png)

每次背叛都有一个完美时刻，无论选硬币的正面还是反面，另一面都是拯救。

---

# 整体评价

T3是Java爽题, TreeSet真的是神器，T4是经典的单调栈应用题, 单调栈往往和贡献思想有一定的关联，最后小小的贪心就好, 非常的板。

T3添加分块的解法，感觉这个思路更直观。

T4补充链式并查集求贡献的方法。

---

## [A. 数组中的最大数对和](https://leetcode.cn/problems/max-pair-sum-in-an-array/)

可以按最大值进行分组, 然后求组内最大值

注：需要小心 不存在的情况

```java
class Solution {
    public int maxSum(int[] nums) {
        // 先分组
        Map<Integer, List<Integer>> hash = new HashMap<>();
        for (int v: nums) {
            int k = String.valueOf(v).chars().max().getAsInt();
            hash.computeIfAbsent(k, x -> new ArrayList<>()).add(v);
        }

        int ans = -1;
        for (var e: hash.entrySet()) {
            List<Integer> tx = e.getValue();
            if (tx.size() >= 2) {
                tx.sort(Comparator.comparing(x -> -x));
                ans = Math.max(ans, tx.get(0) + tx.get(1));
            }
        }
        return ans;
    }
}
```

---

## [B. 翻倍以链表形式表示的数字](https://leetcode.cn/problems/double-a-number-represented-as-a-linked-list/)

贴一个面试官掀桌子的代码

- 额外用了其他数据结构
- 用了大数类

```java
import java.math.BigInteger;

class Solution {
    public ListNode doubleIt(ListNode head) {
        
        // 链表转数组
        StringBuilder sb = new StringBuilder();
        ListNode cur = head;
        while (cur != null) {
            sb.append((char)(cur.val+'0'));
            cur = cur.next;
        }

        // 大数
        BigInteger bi = new BigInteger(sb.toString());
        bi = bi.multiply(BigInteger.valueOf(2));
        String tx = bi.toString();

        // 数组转链表
        ListNode dummy = new ListNode();
        cur = dummy;
        for (char c: tx.toCharArray()) {
            cur.next = new ListNode(c - '0');
            cur = cur.next;
        }

        return dummy.next;
    }
 }
```

---
## [C. 限制条件下元素之间的最小绝对差](https://leetcode.cn/problems/minimum-absolute-difference-between-elements-with-constraint/)

Java爽题，TreeSet支持floor和ceiling的操作, 这边维护差k的窗口即可。

本质就是求, 满足范围内的

- 绝对值左侧最小值
- 绝对值右侧最小值


```java
class Solution {

    public int minAbsoluteDifference(List<Integer> nums, int x) {

        int ans = Integer.MAX_VALUE;
        
        TreeSet<Integer> ts = new TreeSet<>();
        for (int i = x; i < nums.size(); i++) {
            // 窗口补上
            ts.add(nums.get(i - x));
            
            int v = nums.get(i);
            Integer tmp = Integer.MAX_VALUE;
            
            // 左侧的最小值
            Integer lk = ts.floor(v);
            if (lk != null) {
                tmp = Math.min(tmp, v - lk);
            }
            // 右侧的最小值
            Integer hk = ts.ceiling(v);
            if (hk != null) {
                tmp = Math.min(tmp, hk - v);
            }
            ans = Math.min(ans, tmp);
        }

        return ans;
    }

}
```

---

补充一个分块的写法

分块的话，大概是$O(n \sqrt n)$ 的复杂度

因为值域比较大，可以进行离散化梳理，这样可以让数据规模压缩在$O(10^5)$

```java
class Solution {

    static class Block {

        List<Integer>[]g;
        int nBlock;
        int blockSize;

        Map<Integer, Integer> hash = new HashMap<>();

        public Block(List<Integer> nums) {
            // 先离散化, 因为值域还是有些大, 压缩到 10^5 可以接受
            int ptr = 0;
            for (int v: nums.stream().distinct().sorted().collect(Collectors.toList())) {
                hash.put(v, ptr++);
            }

            this.blockSize = (int)Math.sqrt(ptr);
            this.nBlock = ptr / blockSize + 1;

            this.g = new List[this.nBlock];
            Arrays.setAll(g, x->new ArrayList<>());

        }

        int lowBound(int v) {
            int id = hash.get(v);
            int nid = id / blockSize;

            int ans = -10_0000_0000; // 负边界，-inf会溢出
            boolean found = false;
            for (int tv: g[nid]) {
                if (tv <= v) {
                    ans = Math.max(ans, tv);
                    found = true;
                }
            }

            // 命中某一块，就可以直接退出了，因为块间保序
            for (int i = nid - 1; !found && i >= 0; i--) {
                for (int tv: g[i]) {
                    ans = Math.max(ans, tv);
                    found = true;
                }
            }

            return ans;
        }

        int highBound(int v) {
            int id = hash.get(v);
            int nid = id / blockSize;

            int ans = Integer.MAX_VALUE; 
            boolean found = false;
            for (int tv: g[nid]) {
                if (tv >= v) {
                    ans = Math.min(ans, tv);
                    found = true;
                }
            }

            // 命中某一块，就可以直接退出了，因为块间保序
            for (int i = nid + 1; !found && i < g.length; i++) {
                for (int tv: g[i]) {
                    ans = Math.min(ans, tv);
                    found = true;
                }
            }
            return ans;
        }

        void add(int v) {
            int id = hash.get(v);
            int nid = id / blockSize;
            // 这边的去重，很重要，但是力扣的case太随机了，不加更优
            for (int tv: g[nid]) {
                if (tv == v) {
                    return;
                }
            }
            g[nid].add(v);
        }
    }

    public int minAbsoluteDifference(List<Integer> nums, int x) {
        Block block = new Block(nums);

        int ans = Integer.MAX_VALUE;
        for (int i = x; i < nums.size(); i++) {
            block.add(nums.get(i - x));
            int v = nums.get(i);
            int low = block.lowBound(v);
            int high = block.highBound(v);

            int tmp = Math.min(Math.abs(v - low), Math.abs(v - high));
            ans = Math.min(ans, tmp);
        }

        return ans;
    }

}
```



---

## [D. 操作使得分最大](https://leetcode.cn/problems/apply-operations-to-maximize-score/)

因为要求乘积最大，所以核心思路是贪心，先挑最大的数乘积。

因为这边的数，其实是区间代表，也就是时候这个区间内，它被选中了。

那现在的问题回归到本质

> 这个数能成为多少的区间代表。

而这个区间代表，和质数个数/位置有关

因此借助单调栈，把每个数的代表区间范围求出来。

构建递减的单调栈(左侧非递增，右侧严格递减)

计算求的每个数的 $left, right$

那这个数的区间个数为

> $(i - left + 1) * (right - i + 1)$

注意：这边有个long溢出，要小心

最后贪就完事了。

这题很综合，应该涉及到多个知识点
- 单调栈
- 快速幂
- 组合数学
- 贪心
- 贡献

虽然板，但绝对是好题

当然这边还有一些知识点，比如质因子求解

- 可以预处理，魔改筛法，$O(nlog(n))$
- 在线求解，$O(n * \sqrt{v})$

```java
    class Solution {

        static boolean gOk;
        static int MAX_SIZE = 10_0000;
        static int[] gPrimes = new int[MAX_SIZE + 1];

        // 预处理，魔改筛表
        public static void gInitialize() {
            // 模拟筛表，求得1e5范围内的质因子个数
            if (gOk) return;
            boolean[] used = new boolean[MAX_SIZE + 1];
            for (int i = 2; i <= MAX_SIZE; i++) {
                if (!used[i]) {
                    for (int j = i; j <= MAX_SIZE; j+=i) {
                        gPrimes[j]++;
                        used[j] = true;
                    }
                }
            }
            gOk = true;
        }

        public int maximumScore(List<Integer> nums, int k) {
            // 筛表预处理
            gInitialize();

            int n = nums.size();

            Deque<int[]> deq = new ArrayDeque<>();
            deq.offer(new int[] {Integer.MAX_VALUE, -1});

            int[] left = mono(nums, true);
            int[] right = mono(nums, false);

            Integer[] idxs = IntStream.range(0, n).boxed().toArray(Integer[]::new);
            Arrays.sort(idxs, Comparator.comparingInt(x -> -nums.get(x)));

            long nk = k;
            long mod = 10_0000_0007l;

            long ans = 1l;
            for (int i = 0; nk > 0 && i < idxs.length; i++) {
                int id = idxs[i];
                int gv = nums.get(id);
                long range = (long)(id - left[id] + 1) * (right[id] - id + 1);

                long mk = Math.min(nk, range);
                ans = ans * qpow(gv, mk, mod) % mod;
                nk -= mk;
            }

            return (int)(ans % mod);
        }

        // 单调栈
        int[] mono(List<Integer> nums, boolean flag) {
            int n = nums.size();
            int[] res = new int[n];

            if (flag) {
                Deque<int[]> deq = new ArrayDeque<>();
                deq.offer(new int[] {Integer.MAX_VALUE, -1}); // 哨兵
                for (int i = 0; i < n; i++) {
                    int v = nums.get(i);
                    int nv = gPrimes[v];

                    while (!deq.isEmpty() && deq.peekLast()[0] < nv) {
                        deq.pollLast();
                    }
                    res[i] = deq.peekLast()[1] + 1;
                    deq.addLast(new int[] {nv, i});
                }
            } else {
                Deque<int[]> deq = new ArrayDeque<>();
                deq.offer(new int[] {Integer.MAX_VALUE, n}); // 哨兵
                for (int i = n - 1; i >= 0; i--) {
                    int v = nums.get(i);
                    int nv = gPrimes[v];

                    while (!deq.isEmpty() && deq.peekLast()[0] <= nv) {
                        deq.pollLast();
                    }
                    res[i] = deq.peekLast()[1] - 1;
                    deq.addLast(new int[] {nv, i});
                }
            }
            return res;
        }

        // 快速幂
        long qpow(long base, long v, long mod) {
            long res = 1l;
            base %= mod;
            while (v > 0) {
                if ((v & 0x01) == 1) {
                    res = res * base % mod;
                }
                base = base * base % mod;
                v = v >> 1;
            }
            return res;
        }

    }

```

---

补充一个 **链式并查集** 求贡献的思路

因为质因子个数多+位置靠前，是其占区间主导的标准。

可以按每个元素的质因子个数+位置排序(质因子数从小到大，位置从大到小)，然后向左右侧定向合并。那它所覆盖的左右侧最远值，就是它的影响范围.

这边需要引入两个定向并查集dsu1，dsu2，一个专维护左侧，一个专维护右侧。

${scores[i]=(i-dsu1.leader(i)) \times (dsu2.leader(i) - i)}$

链式并查集为了方便处理，把数组整体偏移一位，同时引入$0，n+1$两点作为哨兵。


```java
import java.math.*;

class Solution {

    static int MAX_SIZE = 10_0000;
    static int[] gPrimes = new int[MAX_SIZE + 1];

    // 预处理，魔改筛表
    static {
        // 模拟筛表，求得1e5范围内的质因子个数
        for (int i = 2; i <= MAX_SIZE; i++) {
            if (gPrimes[i] == 0) {
                for (int j = i; j <= MAX_SIZE; j += i) {
                    gPrimes[j]++;
                }
            }
        }
    }

    static class Dsu {
        int n;
        int[] arr;

        public Dsu(int n) {
            this.n = n;
            this.arr = new int[n + 1];
            Arrays.fill(arr, -1);
        }

        int leader(int u) {
            if (arr[u] == -1) return u;
            return arr[u] = leader(arr[u]);
        }

        // 定向合并
        void merge(int u, int v) {
            int uf = leader(u);
            int vf = leader(v);
            if (uf != vf) {
                arr[uf] = vf;
            }
        }
    }

    public int maximumScore(List<Integer> nums, int k) {

        int n = nums.size();

        long mod = 10_0000_0007l;

        List<int[]> arr = new ArrayList<>();
        List<int[]> brr = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            arr.add(new int[]{i, gPrimes[nums.get(i)]});
            brr.add(new int[]{i, nums.get(i)});
        }
        Collections.sort(arr, Comparator.comparingInt((int[] a) -> a[1]).thenComparingInt(a -> -a[0]));
        Collections.sort(brr, Comparator.comparing(x -> -x[1]));

        // 1. 链式并查集求影响范围(左右边界)
        // 整体做一个偏移, 引入哨兵0, n + 1
        Dsu dsu1 = new Dsu(n + 2);
        Dsu dsu2 = new Dsu(n + 2);
        long[] scores = new long[n];
        for (int[] item : arr) {
            int idx = item[0];
            // 定向合并
            dsu1.merge(idx + 1, idx);
            dsu2.merge(idx + 1, idx + 2);

            long l = idx - dsu1.leader(idx + 1) + 1;
            long r = dsu2.leader(idx + 1) - idx - 1;

            scores[idx] = l * r;
        }

        // 2. 贪心解
        long ans = 1l;
        for (int i = 0; k > 0 && i < brr.size(); i++) {
            int idx = brr.get(i)[0], gv = brr.get(i)[1];
            int delta = (int)Math.min(k, scores[idx]);
            // 利用大数类，求快速幂
            ans = ans * BigInteger.valueOf(gv).modPow(BigInteger.valueOf(delta), BigInteger.valueOf(mod)).longValue() % mod;
            k -= delta;
        }

        return (int) (ans % mod);
    }


}


```

---

# 写在最后

如果内心的爱走不出去， 别人的爱也进不来

![image.png](https://pic.leetcode.cn/1691899512-RrxyNR-image.png)
