

# 牛客周赛 Round 28 解题报告 | 珂学家 | 组合数学 + 离散化&树状数组

---

# 前言


---

# 整体评价

还是E稍微有点意思，新周赛好像比预期要简单一些, ^_^.

---

## 欢迎关注

[珂朵莉 牛客周赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=09oWoj)

[珂朵莉 牛客小白月赛专栏](https://www.nowcoder.com/issue/tutorial?zhuanlanId=0pyBbm)

---

## [A. 小红的新周赛](https://ac.nowcoder.com/acm/contest/73239/A)

思路: 模拟

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    int res = 0;
    for (int i = 0; i < 6; i++) {
        int v;
        cin >> v;
        res += v;
    }
    cout << res << endl;
    return 0;
}
```

## [B. 小红的字符串](https://ac.nowcoder.com/acm/contest/73239/B)

思路: 计数+模拟

引入 26 * 26的状态，进行计数

这有个好处，就是天然排序，避免大内存存字符串并排序

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    
    // 26 * 26天然保序
    int cnt[26][26] = {0};
    
    string s;
    cin >> s;
    int n = s.length();
    for (int i = 0; i < n - 1; i++) {
        int p1 = s[i] - 'a';
        int p2 = s[i + 1] - 'a';
        cnt[p1][p2]++;
    }
    
    for (int i = 0; i < 26; i++) {
        for (int j = 0; j < 26; j++) {
            string ts = "";
            ts.push_back((char)(i + 'a'));
            ts.push_back((char)(j + 'a'));
            for (int t = 0; t < cnt[i][j]; t++) {
                cout << ts << endl;
            }
        }
    }
    
}
```

---

## [C. 小红的炸砖块](https://ac.nowcoder.com/acm/contest/73239/C)

思路: 模拟

引入保存每列高度的数组，然后模拟即可

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    
    int n, m, k;
    cin >> n >> m >> k;
    
    vector<int> cols(m, n);
    for (int i = 0; i < k; i++) {
        int r, c;
        cin >> r >> c;
        if (cols[c - 1] >= n - r + 1) {
            cols[c - 1]--;
        }
    }
    
    for (int i = 0; i < n; i++) {
        string r;
        for (int j = 0; j < m; j++) {
            r.push_back(cols[j] >= n - i ? '*' : '.');
        }
        cout << r << endl;
    }
    
    return 0;
}
```

---

## [D. 小红统计区间（easy）](https://ac.nowcoder.com/acm/contest/73239/D)

思路: 滑窗

非常典的一道滑窗题，双指针维护即可

```c++ []
#include <bits/stdc++.h>

using namespace std;

using int64 = long long;

int main() {
    
    int n;
    int64 k;
    cin >> n >> k;
    
    vector<int64> pre(n + 1, 0);
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
        pre[i + 1] = pre[i] + arr[i];
    }
    
    int64 res = 0LL;
    int j = 0;
    for (int i = 0; i < n; i++) {
        while (j <= i && pre[i + 1] - pre[j] >= k) {
            j++;
        }
        res += j;
    }
    cout << res << endl;
    
    return 0;
}
```

----

## [E. 小红的好数组](https://ac.nowcoder.com/acm/contest/73239/E)

思路: 找规律 + 组合数学

case给的非常良心

可以分类讨论，大概有4种类似的序列

arr1 = [偶数，偶数，偶数，偶数，偶数，偶数， ...]

arr2 = [奇数，奇数，偶数，奇数，奇数，偶数，...]

arr3 = [奇数，偶数，奇数，奇数，偶数，奇数，...]

arr4 = [偶数，奇数，奇数，偶数，奇数，奇数，...]

奇数/偶数的分布，呈现强烈的规律

最终为这4种情况的组合方案和

```c++ []
#include <bits/stdc++.h>

using namespace std;

using int64 = long long;

const int64 mod = (long)1e9 + 7;

int64 ksm(int64 b, int64 v) {
    int64 r = 1LL;
    while (v > 0) {
        if (v % 2 == 1) {
            r = r * b % mod;
        }
        v /= 2;
        b = b * b % mod;
    }
    return r;
}

int main() {
    
    int n, k;
    cin >> n >> k;
    
    int k2 = k / 2, k1 = k - k2;
    
    int64 r1 = ksm(k2, n);
    int64 r2 = ksm(k2, n/3) * ksm(k1, n - n/3) % mod;
    int64 r3 = ksm(k2, (n+1)/3) * ksm(k1, n - (n+1)/3) % mod;
    int64 r4 = ksm(k2, (n+2)/3) * ksm(k1, n - (n+2)/3) % mod;
    
    int64 res = (r1 + r2 + r3 + r4) % mod;
    cout << res << endl;
    
    return 0;
}
```

---

## [F. 小红统计区间（hard）](https://ac.nowcoder.com/acm/contest/73239/F)

思路: 离散化+树状数组

也是一道非常典的题

因为存在负数，所以滑窗的基础已经被破坏了

```java []
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

public class Main {

    static class BIT {
        int n;
        int[] arr;
        public BIT(int n) {
            this.n = n;
            this.arr = new int[n + 1];
        }

        int query(int p) {
            int res = 0;
            while (p > 0) {
                res += arr[p];
                p -= p & -p;
            }
            return res;
        }

        void update(int p, int d) {
            while (p <= n) {
                arr[p] += d;
                p += p & -p;
            }
        }
    }

    public static void main(String[] args) {
        AReader sc = new AReader();

        int n = sc.nextInt();
        long k = sc.nextLong();

        long[] arr = new long[n];
        long[] pre = new long[n + 1];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextLong();
            pre[i + 1] = pre[i] + arr[i];
        }

        // 进行离散化
        TreeSet<Long> ids = new TreeSet<>();
        for (long v: pre) {
            ids.add(v);
        }
        int ptr = 0;
        TreeMap<Long, Integer> hp = new TreeMap<>();
        for (long kv: ids) {
            hp.put(kv, ++ptr);
        }

        BIT bit = new BIT(ptr);
        bit.update(hp.get(0l), 1);

        long res = 0;
        for (int i = 0; i < n; i++) {
            long p = pre[i + 1];
            // p - x >= k
            // x <= p - k
            Map.Entry<Long, Integer> ent = hp.floorEntry(p - k);
            if (ent != null) {
                res += bit.query(ent.getValue());
            }
            bit.update(hp.get(p), 1);
        }

        System.out.println(res);

    }

    static
    class AReader {
        private BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        private StringTokenizer tokenizer = new StringTokenizer("");
        private String innerNextLine() {
            try {
                return reader.readLine();
            } catch (IOException ex) {
                return null;
            }
        }
        public boolean hasNext() {
            while (!tokenizer.hasMoreTokens()) {
                String nextLine = innerNextLine();
                if (nextLine == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(nextLine);
            }
            return true;
        }
        public String nextLine() {
            tokenizer = new StringTokenizer("");
            return innerNextLine();
        }
        public String next() {
            hasNext();
            return tokenizer.nextToken();
        }
        public int nextInt() {
            return Integer.parseInt(next());
        }

        public long nextLong() {
            return Long.parseLong(next());
        }

//        public BigInteger nextBigInt() {
//            return new BigInteger(next());
//        }
        // 若需要nextDouble等方法，请自行调用Double.parseDouble包装
    }

}
```

```c++ []
#include <bits/stdc++.h>

using namespace std;
using int64 = long long;

class BIT {
private:
    int n;
    vector<int> arr;
public:
    BIT(int n): n(n), arr(n + 1, 0) {
    }

    int query(int p) {
        int r = 0;
        while (p > 0) {
            r += arr[p];
            p -= p & -p;
        }
        return r;
    }

    void update(int p, int d) {
        while (p <= n) {
            arr[p] += d;
            p += p & -p;
        }
    }
};

int main() {

    int n;
    int64 k;
    cin >> n >> k;

    vector<int64> pre(n + 1, 0LL);
    for (int i = 0; i < n; i++) {
        int v;
        cin >> v;
        pre[i + 1] = pre[i] + v;
    }

    set<int64> ts;
    for (int64 v: pre) {
        ts.insert(v);
    }
    int ptr = 0;
    map<int64, int> idMap;
    for (int64 v: ts) {
        idMap[v] = ++ptr;
    }

    int64 res = 0;
    BIT bit(ptr);
    bit.update(idMap[0], 1);
    for (int i = 0; i < n; i++) {
        int64 p = pre[i + 1];
        if (idMap.find(p - k) != idMap.end()) {
            res += bit.query(idMap[p - k]);
        } else {
            auto iter = idMap.lower_bound(p - k);
            if (iter != idMap.end()) {
                res += bit.query(iter->second - 1);
            } else {
                res += bit.query(ptr);
            }
        }
        bit.update(idMap[p], 1);
    }
    cout << res << endl;

    return 0;
}
```

---

# 写在最后

