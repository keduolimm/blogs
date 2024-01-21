

--- 
# 前言

---

# 整体评价



---

## [A. ACCEPT](https://ac.nowcoder.com/acm/contest/72980/A)

思路: 链条最薄弱的一环决定其强度

取 min{cnt['A'], cnt['C'] / 2, cnt['E'], cnt['P'], cnt['T']}的最小值

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {

    ios::sync_with_stdio(false);
    cout.tie(nullptr);
    
    int t;
    cin >> t;
    while (t-- > 0) {
        int n;
        cin >> n;
        string s;
        cin >> s;
        int hash[26] = {0};
        for (char c: s) {
            hash[c - 'A']++;
        }
        
        int m = min({hash['A' - 'A'], hash['C' - 'A'] / 2,
             hash['E' - 'A'], hash['P' - 'A'], hash['T' - 'A']});
        cout << m << endl;
        
    }
    return 0;
}
```

---
## [B. 咕呱蛙](https://ac.nowcoder.com/acm/contest/72980/B)

思路: 找规律

其 0, 3, 4, 7, 8, 11, 12, ....

其实为 

- f(n) = 2 * n,  n为偶数
- f(n) = 2 * n + 1, n为奇数

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    
    long long n;
    cin >> n;
    
    if (n % 2 == 1) {
        cout << (2 * n + 1) << endl;
    } else {
        cout << 2 * n << endl;
    }
    
    return 0;
}
```

---

## [C. 得分显示](https://ac.nowcoder.com/acm/contest/72980/C)

思路: 二分

经典二分思路

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    
    int n;
    cin >> n;
    int mv = 0;
    vector<int> arr(n, 0);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
        mv = max(mv, arr[i]);
    }
    
    double l = 0;
    double r = mv + 1;
    
    function<bool(double)> checker = [&](double m) {
        double acc = 0;
        for (int i= 0; i < n; i++) {
            acc += m;
            if (floor(acc) > arr[i]) return false;
        }
        return true;
    };
    
    while (r - l >= 1e-6) {
        double m = (l + r) / 2.0;
        if (checker(m)) {
            l = m;
        } else {
            r = m;
        }
    }
    
    printf("%.6f\n", r);
    
    return 0;
}
```

---

## [D. 阿里马马与四十大盗](https://ac.nowcoder.com/acm/contest/72980/D)

思路: 贪心

有缺失的时候，立马补上，这样整体的代价越小

当然这边有个尾巴，需要额外的处理下

```c++ []
#include <bits/stdc++.h>

using namespace std;

int main() {
    
    int n, m;
    cin >> n >> m;
    vector<int> arr(n);
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }
    
    long long acc = 0;
    vector<long long> vec;
    int j = 0;
    while (j < n) {
        if (arr[j] == 0) {
            j++;
        } else {
            int k = j + 1;
            long long tmp = arr[j];
            while (k < n && arr[k] > 0) {
                tmp += arr[k];
                k++;
            }
            vec.push_back(tmp);
            j = k;
            acc += tmp;
        }
    }
    
    bool ok = true;
    
    long long res = n - 1;
    int tmp = m;
    for (int v: vec) {
        if (v >= m) {
            ok = false;
            break;
        }
        if (tmp > acc) {
            break;
        }
        res += (m - tmp);
        tmp = m - v;
        acc -= v;
    }
    if (!ok) cout << "NO" << endl;
    else cout << res << endl;
    return 0;
}
```

---

## [E. 烙饼](https://ac.nowcoder.com/acm/contest/72980/E)

思路: 构造题

其实这题很简单，其限制为

max(最大值, 平均值)

然后在构造的时候，每一个饼最多在2个机器存在，且其长度不会有重叠

```c++ []
#include <bits/stdc++.h>

using namespace std;

using int64=long long;

int main() {
    
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);
    
    
    int n, m;
    cin >> n >> m;
    
    int64 acc = 0;
    vector<int> arr(n);
    for (int i = 0; i < n;i++) {
        cin >> arr[i];
        acc += arr[i];
    }
    
    int64 k1 = (acc + m - 1) / m;
    int64 k2 = *max_element(arr.begin(), arr.end());
    
    int64 k = max(k1, k2);
    
    vector<array<int64, 4>> res;
    
    int slot = 0;
    int64 tmp = 0;
    for (int i = 0; i < n; i++) {
        if (tmp + arr[i] <= k) {
            res.push_back({i + 1, slot + 1, tmp, tmp + arr[i]});

            if (tmp + arr[i] == k) {
                tmp = 0;
                slot++;
            } else {
                tmp = tmp + arr[i];
            }
        } else {
            res.push_back({i + 1, slot + 1, tmp, k});
            tmp = tmp + arr[i] - k;
            slot++;
            res.push_back({i + 1, slot + 1, 0, tmp});
        }
    }
    
    cout << res.size() << '\n';
    for (auto [task, s, l, r] : res) {
        cout << task << " " << s << " " << l << " " << r << "\n";
    }
    
    return 0;
}
```

---

## [F. 龙吸水（EASY）](https://ac.nowcoder.com/acm/contest/72980/F)

---
## [G. 	龙吸水（HARD）](https://ac.nowcoder.com/acm/contest/72980/G)


---

# 写在最后

