# 第 371 场周赛 解题报告 | 珂学家 | 可撤销的0-1Trie



---

# 前言

![image.png](https://pic.leetcode.cn/1699760057-iPXTDf-image.png)


---

# 整体评价

T4是可撤销的0-1Trie，好像是三叶姐姐的经典板子题。异或求最大，一般是用0-1Trie来现实，前几天的每日一题也出现过。

---

## [T1. 找出强数对的最大异或值 I](https://leetcode.cn/contest/weekly-contest-371/problems/maximum-strong-pair-xor-i/)


和T4一起讲

---

## [T2. 高访问员工](https://leetcode.cn/contest/weekly-contest-371/problems/high-access-employees/)

模拟就行

切记不能滑窗，会WA

```java
class Solution {
    
    public List<String> findHighAccessEmployees(List<List<String>> access_times) {
        
        Map<String, List<Integer>> hash = new HashMap<>();
        for (var e: access_times) {
            String n = e.get(0);
            String t = e.get(1);
            int time = Integer.valueOf(t.substring(0, 2)) * 60 + Integer.valueOf(t.substring(2));
            hash.computeIfAbsent(n, x -> new ArrayList<>()).add(time);
        }
        
        List<String> result = new ArrayList<>();
        for (var kv: hash.entrySet()) {
            var list = kv.getValue();
            Collections.sort(list);
            
            boolean ok = false;
            for (int i = 0; i < list.size() - 2; i++) {
                int v1 = list.get(i), v2 = list.get(i + 1), v3 = list.get(i + 2);
                if (v3 - v1 < 60) {
                    ok = true;
                    break;
                }
            }
            if (ok) result.add(kv.getKey());
        }
        return result;
        
    }
    
}
```

---

## [T3. 最大化数组末位元素的最少操作次数](https://leetcode.cn/contest/weekly-contest-371/problems/minimum-operations-to-maximize-last-elements-in-arrays/)


这题就是枚举a, b数组的最后两个元素

然后去验证，在此场景下，需要多少操作次数

```java
class Solution {
    
    static int inf = 0x3f3f3f3f;

    int solve(int[] nums1, int[] nums2, int v1, int v2) {
        int n = nums1.length;
        int cost = 0;
        for (int i = 0; i < n - 1; i++) {
            if (nums1[i] <= v1 && nums2[i] <= v2) continue;
            if (nums1[i] <= v2 && nums2[i] <= v1) cost += 1;
            else return inf;
        }
        return cost;
    }
    
    public int minOperations(int[] nums1, int[] nums2) {
        
        int n = nums1.length;
        int c1 = solve(nums1, nums2, nums1[n - 1], nums2[n - 1]);
        int c2 = solve(nums1, nums2, nums2[n - 1], nums1[n - 1]) + 1;
        
        int r = Math.min(c1, c2);
        
        return r == inf ? -1 : r;
    }
    
}
```



---

## [T4. 找出强数对的最大异或值 II](https://leetcode.cn/contest/weekly-contest-371/problems/maximum-strong-pair-xor-ii/)

0-1 Trie 求最大，已经成为一个经典题目了。

那这题的关键是什么呢？

$$ |y - x| <= min(x, y) $$

$$假设y\ge x, 那么 y - x \le x$$

$$x \le y \le 2 * x$$

这个转换非常的重要

如何维护这个 $x \le y \le 2 * x$呢

- 排序
- 引入双端队列做滑窗
- 引入可撤销的0-1 Trie(引用计数，软删除)

这样就非常的自然，如果有板子的话，就可以秒杀了。

```java
class Solution {

    static class Trie01 {

        static int N = 20;

        Trie01[] cs = new Trie01[2];
        int rel = 0; // 引用计数，软删除
        boolean end;

        public static void addTrie(Trie01 root, int v) {
            Trie01 cur = root;
            for (int i = N - 1; i >= 0; i--) {
                int t = (v >> i) & 0x01;
                if (cur.cs[t] == null) {
                    cur.cs[t] = new Trie01();
                }
                cur.rel++;
                cur = cur.cs[t];
            }
            cur.rel++;
            cur.end = true;
        }

        public static void removeTrie(Trie01 root, int v) {
            Trie01 cur = root;
            for (int i = N - 1; i >= 0; i--) {
                int t = (v >> i) & 0x01;
                cur.rel--;
                cur = cur.cs[t];
            }
            cur.rel--;
            cur.end = true;
        }

        public static int queryMax(Trie01 root, int v, int depth) {
            if (root.end) {
                return 0;
            }
            int t = (v >> depth) & 0x01;
            int tx = 1 ^ t;
            if (root.cs[tx] != null && root.cs[tx].rel > 0) {
                return (1 << depth) | queryMax(root.cs[tx], v, depth - 1);
            }
            return queryMax(root.cs[t], v, depth - 1);
        }

    }

    // x <= y <= 2 * x
    public int maximumStrongPairXor(int[] nums) {
        Arrays.sort(nums);
        Deque<Integer> deq = new ArrayDeque<>();

        int j = 0;
        int ans = 0;
        Trie01 root = new Trie01();
        for (int i = 0; i < nums.length; i++) {
            // 保证y <= 2 * x
            while (j < nums.length && nums[i] * 2 >= nums[j]) {
                Trie01.addTrie(root, nums[j]);
                deq.addLast(nums[j]);
                j++;
            }
            while (!deq.isEmpty() && deq.peekFirst() * 2 < nums[i]) {
                int v = deq.pollFirst();
                Trie01.removeTrie(root, v);
            }

            int tmp = Trie01.queryMax(root, nums[i], Trie01.N - 1);
            ans = Math.max(ans, tmp);
        }
        return ans;
    }

}
```

---

# 写在最后

![image.png](https://pic.leetcode.cn/1699760024-eBDVpP-image.png)
