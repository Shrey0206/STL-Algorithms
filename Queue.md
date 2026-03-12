# Queue and Deque 

## 1. What is it

Both are linear data structures where elements are added and removed at the ends, not in the middle.

| | `queue` | `deque` (double ended queue) |
|---|---|---|
| Header | `<queue>` | `<deque>` |
| Principle | **FIFO (First In First Out)**, the first element inserted is the first to be removed | Insert and remove at **both ends** |
| Add | back only | front and back |
| Remove | front only | front and back |
| Index access `[i]` | No | Yes, O(1) |
| Loop / iterators (`begin()`) | No | Yes |
| `clear()` | No | Yes |
| Type | Container adapter (sits on top of a `deque` by default) | Real container (array of fixed-size blocks) |
| Memory | Same as the container under it | Not one contiguous block, so `&dq[0] + i` is not `&dq[i]` |
| Extra powers | None, only the queue operations | `insert` / `erase` in the middle, `sort`, `resize`, `assign` |

Rule of thumb: use `queue` for BFS and plain FIFO work. Switch to `deque` when you need to add / remove at the front too (0-1 BFS, sliding window maximum), loop over the elements, or access by index.

Headers: `<queue>`, `<deque>` (needed even if you only use `queue` with a `deque` inside)

In the snippets: `q` = `queue<int>` (the baseline, `qp` = `queue<pair<int,int>>`, `qv` = `queue<vector<int>>`), `dq` = `deque<int>` (`dp` = `deque<pair<int,int>>`, `d2` = `deque<deque<int>>`), `x` = a value, `i` = an index, `it` = an iterator.

## 2. CRUD

### Create

```cpp
queue<int> q;                           // empty
queue<int> q2(q);                       // copy of q (or q2 = q)
queue<pair<int,int>> qp;                // queue of pairs
queue<vector<int>> qv;                  // queue of vectors, each element is a whole vector
queue<int, list<int>> ql;               // use list as the underlying container
// no list initialization for queue: queue<int> q = {1, 2, 3}; does NOT work. Push one by one.

deque<int> dq;                          // empty
deque<int> dq(5);                       // size 5, all 0
deque<int> dq(5, -1);                   // size 5, all -1
deque<int> dq = {1, 2, 3};              // with values
deque<int> dq2(dq);                     // copy of dq (or dq2 = dq)
deque<int> dq3(v.begin(), v.end());     // from a vector (or any range)
deque<pair<int,int>> dp;                // deque of pairs
```

### Insert

```cpp
q.push(x);                              // add x at the back
q.emplace(x);                           // same, builds the element in place (good for pairs / structs)
qp.emplace(1, 2);                       // no need to write make_pair
qp.push({1, 2});                        // list syntax also works
qv.push({1, 2, 3});                     // push a vector
qv.emplace(3, 0);                       // builds vector<int>(3, 0) = {0, 0, 0} in place

dq.push_back(x);                        // add at the back
dq.push_front(x);                       // add at the front
dq.emplace_back(x);                     // same, builds in place
dq.emplace_front(x);
dp.emplace_back(1, 2);                  // no need to write make_pair

dq.insert(dq.begin() + 2, x);           // insert x at index 2, O(n)
dq.insert(dq.begin(), 3, x);            // insert 3 copies of x at the front
dq.insert(dq.end(), v.begin(), v.end());// append a range
```

Insert at the front or the back is O(1). Insert in the middle (deque only) is O(n).

### Read

```cpp
q.front();                              // first element (next to be removed), does NOT remove it
q.back();                               // last element (most recently added)
q.size();   q.empty();
if (!q.empty()) cout << q.front();      // always check before front() / back()
auto [a, b] = qp.front();               // structured binding on a pair
qv.front()[0];                          // first element of the front vector

dq[i];                                  // element at index i, no bounds check
dq.at(i);                               // same, throws out_of_range if i is invalid
dq.front();   dq.back();                // first / last element
dq.size();    dq.empty();

for (int v : dq) cout << v << ' ';      // range-for (front to back)
for (auto &v : dq) v *= 2;              // by reference, to change
for (auto it = dq.rbegin(); it != dq.rend(); ++it) cout << *it;   // reverse
for (auto &[a, b] : dp) cout << a << b; // structured binding on pairs
d2[i][j];                               // 2D access
```

A `queue` shows only the front and the back. To see everything you have to pop.

### Update

```cpp
q.front() = 10;                         // overwrite the front
q.back()++;                             // change the back in place
qv.front()[0] = 9;                      // change one element inside the front vector
qv.back().push_back(4);                 // add to the back vector
// the elements in the middle of a queue are not accessible

dq[i] = 10;                             // overwrite by index
dq.front() = 1;   dq.back() += 5;       // change the ends
dq.assign(5, 0);                        // replace everything: 5 zeros
dq.resize(10);                          // grow (new elements are 0) or shrink
dq.resize(10, -1);                      // grow with -1
swap(dq[i], dq[j]);                     // swap two elements
dq.swap(dq2);                           // swap two whole deques
```

### Delete

```cpp
q.pop();                                // remove the front, returns NOTHING (void)

// read and remove together
int x = q.front();
q.pop();

// clear everything (queue has no clear())
while (!q.empty()) q.pop();
q = queue<int>();                       // or: replace with an empty queue

qv.pop();                               // removes the whole front vector, not one element
vector<int> v = qv.front();             // copy the front vector first if you still need it
qv.pop();

dq.pop_back();                          // remove the last (returns nothing)
dq.pop_front();                         // remove the first (returns nothing)
dq.erase(dq.begin() + 2);               // remove index 2, O(n)
dq.erase(dq.begin(), dq.begin() + 3);   // remove a range [0, 3)
dq.clear();                             // remove everything
dq.erase(remove(dq.begin(), dq.end(), x), dq.end());   // remove all occurrences of x
```

A `queue` removes only from the front. A `deque` removes from both ends.

## 3. Important functions

| Function | What it does | Example |
|---|---|---|
| `push(x)` / `push_back(x)` | add at the back | `q.push(5);` `dq.push_back(5);` |
| `emplace(args)` / `emplace_back` | add at the back, built in place | `qp.emplace(1, 2);` |
| `front()` | read / change the first element | `int f = q.front();` |
| `back()` | read / change the last element | `int b = q.back();` |
| `pop()` / `pop_front()` | remove the first (returns nothing) | `q.pop();` `dq.pop_front();` |
| `size()` | number of elements | `q.size();` |
| `empty()` | true if no elements | `if (q.empty())` |
| `swap` | swap two containers | `q.swap(q2);` |
| `==`, `<` | compare element by element | `if (q == q2)` |

**Only for `deque`, `dq` = `deque<int>`:**

| Function | What it does | Example |
|---|---|---|
| `push_front(x)` / `emplace_front` | add at the front | `dq.push_front(5);` |
| `pop_back()` | remove the last (returns nothing) | `dq.pop_back();` |
| `[i]` / `at(i)` | index access | `dq[2]` |
| `insert(it, x)` | insert before `it` | `dq.insert(dq.begin()+1, 7);` |
| `erase(it)` / `erase(a, b)` | remove one / a range | `dq.erase(dq.begin());` |
| `clear()` | remove all | `dq.clear();` |
| `resize(n)` | change the size | `dq.resize(10);` |
| `assign(n, x)` | replace contents | `dq.assign(5, 0);` |
| `begin()` / `end()` / `rbegin()` / `rend()` | iterators | `for (auto it = dq.begin(); it != dq.end(); ++it)` |

Algorithms that work on a deque (it has random access iterators):

```cpp
sort(dq.begin(), dq.end());                         // sort ascending
sort(dq.rbegin(), dq.rend());                       // sort descending
reverse(dq.begin(), dq.end());                      // reverse
*min_element(dq.begin(), dq.end());                 // minimum
*max_element(dq.begin(), dq.end());                 // maximum
accumulate(dq.begin(), dq.end(), 0LL);              // sum (0LL to avoid overflow)
find(dq.begin(), dq.end(), x);                      // iterator to x, or end()
count(dq.begin(), dq.end(), x);                     // how many x
lower_bound(dq.begin(), dq.end(), x);               // first element >= x (sorted deque)
rotate(dq.begin(), dq.begin() + k, dq.end());       // left rotate by k
```

Passing them to functions:

```cpp
void f(queue<int> &q);                  // by reference: no copy, changes affect the original
void g(queue<int> q);                   // by value: copies the whole queue, O(n)
void h(const deque<int> &dq);           // by const reference: no copy, read only
```

### Using a deque as a queue (when you need to loop or remove from both ends)

```cpp
deque<int> dq;
dq.push_back(x);                        // enqueue
dq.front();                             // front
dq.pop_front();                         // dequeue
dq.clear();                             // clear (queue has no clear)
for (int v : dq) cout << v << ' ';      // print from front to back
```

## 4. Interview and important points

### Complexity

| Operation | `queue` | `deque` |
|---|---|---|
| Push at the back | O(1) | O(1) |
| Push at the front | not available | O(1) |
| Pop from the front | O(1) | O(1) |
| Pop from the back | not available | O(1) |
| `front` / `back` | O(1) | O(1) |
| `dq[i]` / `at(i)` | not available | O(1) |
| `size` / `empty` | O(1) | O(1) |
| `insert` / `erase` in the middle | not available | O(n) |
| Search for a value | O(n), and you have to pop to do it | O(n) |
| `sort` | not available | O(n log n) |
| Copy / `clear` | O(n) | O(n) |

### Commonly asked

1. **What is a queue?** A FIFO structure. Insert at the back, remove from the front.
2. **What is a deque?** A double ended queue. Insert and remove at both ends in O(1), with random access by index.
3. **Why does `pop()` return nothing?** For exception safety. If copying the removed value threw an exception, the element would be lost. So you read with `front()` first and then call `pop()`. Same for `pop_back()` / `pop_front()`.
4. **What happens if you call `front()`, `back()` or `pop()` on an empty queue or deque?** Undefined behavior. Always check `empty()` first.
5. **What does `queue` use inside?** A `deque` by default. You can choose `list` as the second template argument. `vector` does not work because it has no `pop_front()`.
6. **Why can't you iterate over a `queue`?** It is a container adapter that hides everything except the front and the back. Use a `deque` if you need to loop.
7. **Queue vs stack?** Queue is FIFO (add at the back, remove from the front), stack is LIFO (add and remove at the top).
8. **Queue vs `priority_queue`?** `priority_queue` removes the element with the highest priority (max by default), not the oldest. It is a heap with O(log n) push and pop.
9. **Deque vs vector?** Deque is fast at both ends, vector only at the back. Vector is contiguous in memory (faster, cache friendly, can pass `v.data()`), deque is not. `push_front` on a vector is O(n).
10. **Deque vs list?** Deque has O(1) index access and is cache friendlier. List has O(1) insert / erase in the middle once you have the iterator, but no indexing.
11. **How is a deque stored?** A map (array of pointers) to fixed-size blocks. That is why both ends can grow without moving the old elements. `dq[i]` is still O(1), with a bigger constant than `vector` because it first finds the block and then the offset.
12. **Where are queues and deques used?** BFS, level-order tree traversal, task / CPU scheduling, printer queue, request buffering, producer-consumer (queue). Sliding window maximum / minimum, 0-1 BFS, palindrome check, undo / redo with a limit (deque).
13. **What is a circular queue?** A queue built on a fixed array where the back wraps around to the start of the array, so the freed slots at the front are reused. Use `(i + 1) % n` for the index. Needs `front`, `rear`, and `size` (or one empty slot) to tell full from empty.
14. **How do you implement a queue using two stacks?** Push into `in`. When you need to pop or peek and `out` is empty, move everything from `in` to `out`, which reverses the order. Pop from `out`. Amortized O(1).
15. **How do you implement a stack using queues?** After pushing `x` into a queue of size `n`, rotate the queue `n` times (move the front to the back) so `x` ends up at the front.
16. **Why is BFS done with a queue?** A queue processes nodes in the order they were discovered, so nodes closer to the source are always handled before farther ones. That gives the shortest path in an unweighted graph.
17. **What is a monotonic deque?** A deque that always stays sorted (increasing or decreasing). Before pushing at the back, you pop everything from the back that breaks the order. The front is the answer. It solves "sliding window maximum / minimum" in O(n) because each element is pushed and popped at most once.
18. **What is 0-1 BFS?** BFS on a graph with edge weights 0 or 1. Push the neighbour at the **front** if the edge weight is 0 and at the **back** if it is 1. Gives shortest paths in O(V + E), faster than Dijkstra.
19. **Deque iterator invalidation?** `insert` / `erase` in the middle invalidates **all** iterators and references. `push_back` / `push_front` invalidate all **iterators** but not references. `pop_back` / `pop_front` invalidate only the removed element.

### Mistakes to look out for

- Calling `front()`, `back()` or `pop()` on an empty queue / deque. Check `!q.empty()`.
- Writing `int x = q.pop();`. It does not compile because `pop*` returns `void`. Use `front()` then `pop()`.
- Trying `q[i]`, a range-for, or `q.begin()` on a `queue`. They do not exist. Use a `deque`.
- Looking for `q.clear()` on a `queue`. It does not exist. Use a `while` loop or assign an empty queue.
- Passing a `queue` or `deque` by value into a function or a recursive call. It copies everything each time.
- Holding `int &r = q.front();` and then calling `pop()`. The reference now points to a removed element.
- Reading `q.size()` in the loop condition while pushing inside the loop (`for (int i = 0; i < q.size(); i++)`). The size keeps changing. Store it in `sz` before the loop.
- In BFS, marking a node visited when you **pop** it instead of when you **push** it. The same node gets pushed many times.
- Using `queue` when you need the smallest or largest element. That is a `priority_queue`.
- `dq.size() - 1` when the deque is empty. `size()` is unsigned, so it wraps to a huge number. Use `(int)dq.size() - 1`.
- Using `dq[i]` with an invalid index. No bounds check. Use `at(i)` while debugging.
- Treating the deque as contiguous memory (passing `&dq[0]` as an array). It is not.
- Erasing inside a range-for on a deque. Use `it = dq.erase(it)` in an iterator loop.
- In the monotonic deque, using `if` instead of `while` when popping from the back, or storing **values** instead of **indexes**. You then cannot tell when an element left the window.
- Using `deque` when you only need the back. A `vector` is faster.

### Common patterns

```cpp
// BFS on a graph, returns shortest distance (in edges) from src, -1 if unreachable
vector<int> bfs(const vector<vector<int>> &g, int src) {
    int n = g.size();
    vector<int> dist(n, -1);
    queue<int> q;
    dist[src] = 0;
    q.push(src);
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        for (int v : g[u])
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;          // mark when pushing
                q.push(v);
            }
    }
    return dist;
}

// level order traversal of a binary tree (take the size before the inner loop)
vector<vector<int>> levelOrder(TreeNode *root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();                      // number of nodes in the current level
        vector<int> level;
        while (sz--) {
            TreeNode *u = q.front();
            q.pop();
            level.push_back(u->val);
            if (u->left) q.push(u->left);
            if (u->right) q.push(u->right);
        }
        res.push_back(level);
    }
    return res;
}

// sliding window maximum (monotonic deque of indexes, values decreasing)
vector<int> maxWindow(const vector<int> &v, int k) {
    vector<int> res;
    deque<int> dq;
    for (int i = 0; i < (int)v.size(); i++) {
        while (!dq.empty() && dq.front() <= i - k) dq.pop_front();      // left the window
        while (!dq.empty() && v[dq.back()] <= v[i]) dq.pop_back();      // can never be the max
        dq.push_back(i);
        if (i >= k - 1) res.push_back(v[dq.front()]);
    }
    return res;
}

// 0-1 BFS, g[u] = {v, w} with w in {0, 1}
vector<int> bfs01(const vector<vector<pair<int,int>>> &g, int src) {
    int n = g.size();
    vector<int> dist(n, INT_MAX);
    deque<int> dq;
    dist[src] = 0;
    dq.push_back(src);
    while (!dq.empty()) {
        int u = dq.front();
        dq.pop_front();
        for (auto [v, w] : g[u])
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.push_front(v);   // free edge: process first
                else dq.push_back(v);
            }
    }
    return dist;
}

// palindrome check with a deque
bool isPal(const string &s) {
    deque<char> dq(s.begin(), s.end());
    while (dq.size() > 1) {
        if (dq.front() != dq.back()) return false;
        dq.pop_front();
        dq.pop_back();
    }
    return true;
}
```

Classic problems built on queues and deques: binary tree level order traversal, rotting oranges, number of islands (BFS), shortest path in a binary matrix, walls and gates, sliding window maximum, implement queue using stacks, implement stack using queues, design circular queue / deque, 0-1 BFS shortest path.
