

[208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)

定义TrieNode节点

```c++ []

class TrieNode {
public:
    friend class Trie;
    TrieNode() {
        for (int i = 0; i < 26; i++) {
            cs[i] = nullptr;
        }
        flag = false;
    }
private:
    TrieNode *cs[26];
    bool flag;
};

class Trie {
public:
    TrieNode *root;
    Trie() {
        root = new TrieNode();
    }
    ~Trie() {
        delete root;
        root = nullptr;
    }
    
    void insert(string word) {
        TrieNode *cur = root;
        for (char c: word) {
            int p = c - 'a';
            if (cur->cs[p] == nullptr) {
                cur->cs[p] = new TrieNode();
            }
            cur = cur->cs[p];
        }
        cur->flag = true;
    }
    
    bool search(string word) {
        TrieNode *cur = root;
        for (char c: word) {
            int p = c - 'a';
            if (cur->cs[p] == nullptr) {
                return false;
            }
            cur = cur->cs[p];
        }
        return cur->flag == true;
    }
    
    bool startsWith(string prefix) {
        TrieNode *cur = root;
        for (char c: prefix) {
            int p = c - 'a';
            if (cur->cs[p] == nullptr) {
                return false;
            }
            cur = cur->cs[p];
        }
        return true;
    }
};

/**
 * Your Trie object will be instantiated and called as such:
 * Trie* obj = new Trie();
 * obj->insert(word);
 * bool param_2 = obj->search(word);
 * bool param_3 = obj->startsWith(prefix);
 */
```