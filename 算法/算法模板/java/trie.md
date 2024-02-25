


# Trie

- 字典树
- 0-1 trie

```java []
class Trie {
    Trie[] cs = new Trie[26];
    int rel;
    
    void add(String w) {
        Trie cur = this;
        for (char c: w.toCharArray()) {
            int p = c - 'a';
            if (cur.cs[p] == null) {
                cur.cs[p] = new Trie();
            }
            cur = cur.cs[p];
        }
        cur.rel++;
    }
    
    // 根据需求添加各种功能函数
}

```

---

# 题单

