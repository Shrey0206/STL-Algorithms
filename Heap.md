# Heap (priority_queue)

## 1. What is it

Heap is a tree based data structure (stored in an array) where the **top element is always the largest** (max heap) or **the smallest** (min heap). In C++ it is available as `priority_queue`.

- You can only: add an element, look at the top, remove the top.
- The top is the element with the highest priority, not the oldest (this is the difference from a normal `queue`).
- No index access (`pq[i]` does not exist), and no looping over it.
- It is a **container adapter**: it sits on top of a `vector` and keeps it arranged as a heap.
- `priority_queue` is a **max heap** by default. For a min heap you pass `greater<int>`.
- Needs `<queue>`.

| | Max heap | Min heap |
|---|---|---|
| Declaration | `priority_queue<int> pq;` | `priority_queue<int, vector<int>, greater<int>> pq;` |
| `top()` gives | largest | smallest |
| Parent vs child | parent >= children | parent <= children |


Headers: `<queue>` (for `priority_queue`), `<vector>`, `<functional>` (for `greater`), `<algorithm>` (for `make_heap`, `push_heap`, `pop_heap`)

In the snippets: `pq` = `priority_queue<int>` (max heap), `mn` = min heap of int, `pp` = `priority_queue<pair<int,int>>`, `x` = a value.

```cpp
typedef priority_queue<int, vector<int>, greater<int>> minpq;    // short name for the min heap
```

## 2. CRUD

### Create

```cpp
priority_queue<int> pq;                                          // empty max heap
priority_queue<int, vector<int>, greater<int>> mn;               // empty min heap
priority_queue<int> pq2(pq);                                     // copy of pq (or pq2 = pq)
priority_queue<pair<int,int>> pp;                                // max heap of pairs
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> mp;   // min heap of pairs

priority_queue<int> pq3(v.begin(), v.end());                     // from a vector, builds in O(n)
priority_queue<int, vector<int>, greater<int>> mn2(v.begin(), v.end());   // min heap from a vector
```

There is no list initialization (`priority_queue<int> pq = {1, 2, 3};` does not work). Push the values one by one, or build from a range.

### Insert (push)

```cpp
pq.push(x);                             // add x, O(log n)
pq.emplace(x);                          // same, builds the element in place
pp.push({5, 2});                        // push a pair
pp.emplace(5, 2);                       // no need to write make_pair

mn.push(-x);                            // trick: push negative values into a max heap to get a min heap
```

### Read

```cpp
pq.top();                               // the largest element (smallest for a min heap), does NOT remove it
pq.size();                              // number of elements
pq.empty();                             // is it empty

if (!pq.empty()) cout << pq.top();      // always check before top()

auto [a, b] = pp.top();                 // structured binding on a pair
pp.top().first;                         // first of the top pair
```

Only the top is visible. To see everything you have to pop (see below).

### Update

`top()` returns a **const** reference, so you cannot change the top in place. To update, remove and insert again.

```cpp
int t = pq.top();
pq.pop();
pq.push(t + 5);                         // "change" the top

// to change an element in the middle: push the new value, and skip the old one when you pop it (lazy deletion)
```

### Delete (pop)

```cpp
pq.pop();                               // remove the top, returns NOTHING (void), O(log n)

// read and remove together
int x = pq.top();
pq.pop();

// clear everything (priority_queue has no clear())
while (!pq.empty()) pq.pop();
pq = priority_queue<int>();             // or: replace with an empty heap

// print in sorted order (destroys the heap)
while (!pq.empty()) { cout << pq.top() << ' '; pq.pop(); }
```

You cannot remove an element from the middle. Use lazy deletion (see Common patterns) or a `set` / `multiset`.

## 3. Important functions

| Function | What it does | Example |
|---|---|---|
| `push(x)` | add an element, O(log n) | `pq.push(5);` |
| `emplace(args)` | add an element, built in place | `pp.emplace(5, 2);` |
| `top()` | read the top (largest, or smallest for a min heap) | `int t = pq.top();` |
| `pop()` | remove the top (returns nothing), O(log n) | `pq.pop();` |
| `size()` | number of elements | `pq.size();` |
| `empty()` | true if no elements | `if (pq.empty())` |
| `swap` | swap two heaps | `pq.swap(pq2);` |

Passing a heap to a function:

```cpp
void f(priority_queue<int> &pq);        // by reference: no copy, changes affect the original
void g(priority_queue<int> pq);         // by value: copies the whole heap, O(n)
```


### Heap with a vector (when you need to loop or change elements)

`make_heap`, `push_heap` and `pop_heap` turn a `vector` into a heap, so you can still loop over it and access any index.

```cpp
vector<int> h = {3, 1, 4, 1, 5};
make_heap(h.begin(), h.end());          // max heap, O(n)
h.push_back(9);
push_heap(h.begin(), h.end());          // fix the heap after push_back, O(log n)
pop_heap(h.begin(), h.end());           // moves the top to the back, O(log n)
int top = h.back();
h.pop_back();                           // now actually remove it
h.front();                              // the top of the heap
sort_heap(h.begin(), h.end());          // sorts ascending, heap is gone after this
is_heap(h.begin(), h.end());            // check if it is still a heap

make_heap(h.begin(), h.end(), greater<int>());    // min heap (use the same comparator in every call)
```

## 4. Interview and important points

### Complexity

| Operation | Time |
|---|---|
| `push` / `emplace` | O(log n) |
| `top` | O(1) |
| `pop` | O(log n) |
| `size` / `empty` | O(1) |
| Build a heap from n elements (range constructor, `make_heap`) | O(n) |
| n pushes one by one | O(n log n) |
| Heap sort | O(n log n) |
| Search for a value / remove from the middle | O(n), not supported by `priority_queue` |
| Copy | O(n) |

### Commonly asked

1. **What is a heap?** A complete binary tree where every parent is >= its children (max heap) or <= its children (min heap). It is stored in an array, so there are no pointers.
2. **Why is it called a priority queue?** Elements come out by priority (largest or smallest first), not by the order they went in.
3. **How do you make a min heap in C++?** `priority_queue<int, vector<int>, greater<int>>`. Or push negative values into the default max heap.
4. **Why does `pop()` return nothing?** For exception safety. If copying the removed value threw an exception, the element would be lost. So you read with `top()` first and then call `pop()`.
5. **What happens if you call `top()` or `pop()` on an empty heap?** Undefined behavior. Always check `empty()` first.
6. **What does `priority_queue` use inside?** A `vector` by default, kept as a heap with `push_heap` / `pop_heap`. `deque` also works.
7. **Why can't you iterate over a `priority_queue`?** It is a container adapter that hides everything except the top. Use a `vector` with `make_heap` if you need to loop.
8. **How does push work?** Add at the end of the array, then move it **up** while it is bigger than its parent (sift up). O(log n).
9. **How does pop work?** Swap the top with the last element, remove the last, then move the new top **down** by swapping with the larger child (sift down). O(log n).
10. **Why is building a heap O(n) and not O(n log n)?** Sift down starts from the bottom. Most nodes are near the leaves and move only a few levels, so the total work adds up to O(n).
11. **Heap vs sorted array vs BST?** Heap: O(1) top, O(log n) push / pop, but no search. Sorted array: O(1) top, O(n) insert. BST / `set`: O(log n) for everything including search and delete, but slower and heavier than a heap.
12. **Heap vs `set` / `multiset`?** `multiset` also gives min / max (`begin()` / `rbegin()`) and can erase any element in O(log n), but it is slower and uses more memory. Use `priority_queue` when you only need the top.
13. **Where are heaps used?** Dijkstra and Prim, top K elements, merge K sorted lists, median of a stream, task scheduling, heap sort, Huffman coding, interval scheduling.
14. **How do you find the K largest elements?** Keep a **min heap** of size K. Push each element, and if the size goes above K, pop the smallest. At the end the heap holds the K largest. O(n log K).
15. **How do you find the median of a stream?** Two heaps: a max heap for the smaller half and a min heap for the larger half. Keep their sizes equal (or differ by 1). The median is one top or the average of both tops.
16. **How do you merge K sorted lists?** Put the head of each list in a min heap. Pop the smallest, push the next element of the same list. O(N log K).
17. **How do you delete an arbitrary element?** `priority_queue` cannot. Use lazy deletion (keep a "deleted" map and skip those on `top()`), or use a `set`.
18. **What is the difference in the comparator compared to `sort`?** `sort` with `less` gives ascending order, `priority_queue` with `less` gives the **largest on top**. So `greater` gives the min heap.
19. **Is a heap stable?** No. Equal elements can come out in any order.

### Mistakes to look out for

- Calling `top()` or `pop()` on an empty heap. Check `!pq.empty()`.
- Writing `int x = pq.pop();`. It does not compile because `pop()` returns `void`. Use `top()` then `pop()`.
- Forgetting that the default is a **max** heap. For Dijkstra and "K largest" you need a **min** heap.
- Writing `priority_queue<int, greater<int>>`. It must be `priority_queue<int, vector<int>, greater<int>>` (all three arguments).
- Trying `pq[i]`, a range-for, or `pq.begin()`. They do not exist.
- Looking for `pq.clear()`. It does not exist. Use a `while` loop or assign an empty heap.
- Trying to change `pq.top()` in place. It is const. Pop and push again.
- Using `>=` or `<=` in a custom comparator. It must be a strict weak ordering (`<` or `>`), otherwise you get undefined behavior or a crash.
- Getting the custom comparator direction backwards. Remember: the element that is "less" by your comparator goes down, the "greatest" is on top.
- Pushing the same node many times in Dijkstra and not skipping the stale entries (`if (d > dist[u]) continue;`).
- Negating `INT_MIN` for the "push negative" trick. It overflows. Use `greater<int>` instead.
- Passing a `priority_queue` by value into a function or a recursive call. It copies the whole heap each time.
- Using `pq.size() - 1` on an empty heap. `size()` is unsigned, so it wraps. Use `(int)pq.size() - 1`.
- Building a heap with n `push` calls when you already have all the data. The range constructor / `make_heap` is O(n).

### Common patterns

```cpp
// K largest elements (min heap of size K), result is the K largest in any order
vector<int> kLargest(const vector<int> &v, int k) {
    priority_queue<int, vector<int>, greater<int>> mn;
    for (int x : v) {
        mn.push(x);
        if ((int)mn.size() > k) mn.pop();       // drop the smallest
    }
    vector<int> res;
    while (!mn.empty()) { res.push_back(mn.top()); mn.pop(); }
    return res;
}

// Kth largest element: the top of the min heap of size K
int kthLargest(const vector<int> &v, int k) {
    priority_queue<int, vector<int>, greater<int>> mn;
    for (int x : v) {
        mn.push(x);
        if ((int)mn.size() > k) mn.pop();
    }
    return mn.top();
}

// Dijkstra, g[u] = {v, w}, returns shortest distances from src
vector<long long> dijkstra(const vector<vector<pair<int,int>>> &g, int src) {
    int n = g.size();
    vector<long long> dist(n, LLONG_MAX);
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<pair<long long,int>>> mn;
    dist[src] = 0;
    mn.push({0, src});                          // {distance, node}
    while (!mn.empty()) {
        auto [d, u] = mn.top();
        mn.pop();
        if (d > dist[u]) continue;              // stale entry, skip
        for (auto [v, w] : g[u])
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                mn.push({dist[v], v});
            }
    }
    return dist;
}

// merge K sorted lists (vectors), O(N log K)
vector<int> mergeK(const vector<vector<int>> &lists) {
    using T = tuple<int,int,int>;               // value, list index, position in that list
    priority_queue<T, vector<T>, greater<T>> mn;
    for (int i = 0; i < (int)lists.size(); i++)
        if (!lists[i].empty()) mn.push({lists[i][0], i, 0});
    vector<int> res;
    while (!mn.empty()) {
        auto [val, i, j] = mn.top();
        mn.pop();
        res.push_back(val);
        if (j + 1 < (int)lists[i].size()) mn.push({lists[i][j + 1], i, j + 1});
    }
    return res;
}

// median of a stream (two heaps)
struct Median {
    priority_queue<int> lo;                                     // max heap, smaller half
    priority_queue<int, vector<int>, greater<int>> hi;          // min heap, larger half
    void add(int x) {
        lo.push(x);
        hi.push(lo.top());                                      // move the biggest of the small half up
        lo.pop();
        if (hi.size() > lo.size()) {                            // keep lo equal or one bigger
            lo.push(hi.top());
            hi.pop();
        }
    }
    double get() {
        if (lo.size() > hi.size()) return lo.top();
        return (lo.top() + hi.top()) / 2.0;
    }
};

// top K frequent elements
vector<int> topKFrequent(const vector<int> &v, int k) {
    unordered_map<int,int> freq;
    for (int x : v) freq[x]++;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> mn;   // {count, value}
    for (auto &[val, c] : freq) {
        mn.push({c, val});
        if ((int)mn.size() > k) mn.pop();
    }
    vector<int> res;
    while (!mn.empty()) { res.push_back(mn.top().second); mn.pop(); }
    return res;
}

// lazy deletion: mark removed values, skip them when they reach the top
unordered_map<int,int> removed;                 // value -> how many times it was "deleted"
void lazyPop(priority_queue<int> &pq) {
    while (!pq.empty() && removed[pq.top()] > 0) {
        removed[pq.top()]--;
        pq.pop();
    }
}

// heap sort with the STL
void heapSort(vector<int> &v) {
    make_heap(v.begin(), v.end());
    sort_heap(v.begin(), v.end());              // ascending
}
```

Classic problems built on heaps: Kth largest element in an array, top K frequent elements, K closest points to origin, merge K sorted lists, find median from data stream, task scheduler, last stone weight, network delay time (Dijkstra), meeting rooms II, reorganize string, sliding window median.
