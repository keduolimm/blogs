

# 离散化模板

离散化用unique(sort())更合适

```c++ []
template <typename T> struct Discrete {
    Discrete() {}
    void add(T t) {
        p.pb(t);
    }
     
    void init() {
        sort(all(p));
        p.resize(unique(all(p)) - p.begin());
    }
     
    int size() {return p.size();}
     
    int id(T t) {
        return lower_bound(all(p), t) - p.begin();
    }
    int id2(T t) {
        return upper_bound(all(p), t) - p.begin() - 1;
    }
    T operator[](int id) {return p[id];}
  
        vector<T> &get() {return p;}
    vector<T> p;
};
```