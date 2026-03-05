# Vector

## 1. What is it

A dynamic array. Same as an array (contiguous memory, index starting from 0, instant access by index), but the size can grow and shrink at runtime. When it runs out of space, it allocates a bigger block (usually double), copies everything over and frees the old one.

- Lives on the heap, so large sizes are safe (no stack overflow).
- Needs `<vector>`.
- Two numbers to know: **size** (elements in use) and **capacity** (space reserved).

Headers: `<vector>`, `<algorithm>` (sort, find...), `<numeric>` (accumulate, iota)

In the snippets: `v` = `vector<int>`, `v2` = another vector, `a` = C array, `n` = size, `i` = index, `l`, `r` = range indexes, `k` = shift count, `x` = a value.

## 2. CRUD

### Create

```cpp
vector<int> v;                     // empty
vector<int> v(5);                  // 5 elements, all 0
vector<int> v(5, 7);               // 5 elements, all 7
vector<int> v = {1, 2, 3};         // Vector with 1,2,3
vector<int> v2(v);                 // copy of v (or v2 = v)
vector<int> v3(v.begin(), v.begin() + 2);   // copy of a range
vector<int> v(a, a + n);           // from a C array

v.reserve(100);                    // reserve space, size stays 0 (no re-allocations while pushing)
```

### Read

```cpp
v[i];                              // access by index, NO bounds check
v.at(i);                           // throws out_of_range if i is invalid
v.front();  v.back();              // first and last
v.size();   v.empty();             // size and is-empty check

for (int x : v) cout << x;         // loop over values
for (int &x : v) x *= 2;           // & lets you change them
for (int i = 0; i < v.size(); i++) cout << v[i];
for (auto it = v.begin(); it != v.end(); ++it) cout << *it;   // using iterator
for (auto it = v.rbegin(); it != v.rend(); ++it) cout << *it;   // reverse iteration
```

### Update

```cpp
v[2] = 10;                         // set one element
swap(v[0], v[1]);                  // swap two elements
v = v2;                            // copy whole vector
v.assign(5, 0);                    // replace contents with 5 zeros
v.resize(10);                      // new size 10 (new elements are 0, or cut if smaller)
fill(v.begin(), v.end(), 7);       // set everything to 7
```

### Insert

```cpp
v.push_back(x);                  // add at the end, O(1) amortized
v.emplace_back(x);               // same, builds the element in place (good for pairs/structs)
v.insert(v.begin() + i, x);      // insert x at index i, O(n)
```

### Delete

```cpp
v.pop_back();                      // remove last, O(1)  (do not call on empty vector)
v.erase(v.begin() + i);            // remove element at index i, O(n)
v.erase(v.begin() + l, v.begin() + r);     // remove range [l, r), r exclusive
v.clear();                         // remove everything (size 0, capacity stays)

// remove all occurrences of value x (erase-remove idiom)
v.erase(remove(v.begin(), v.end(), x), v.end());

// remove duplicates (sort first)
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());
```

## 3. Important functions

Pattern: most algorithms take `(begin, end)`. For a vector that is `(v.begin(), v.end())`. End is **exclusive**.

| Function | What it does | Example |
|---|---|---|
| `push_back` / `pop_back` | add / remove at the end | `v.push_back(5);` |
| `insert` / `erase` | add / remove at a position | `v.erase(v.begin() + i);` |
| `size` / `empty` | element count / is it empty | `if (!v.empty())` |
| `resize` | change the size | `v.resize(n);` |
| `reserve` | reserve capacity in advance | `v.reserve(n);` |
| `capacity` | space currently reserved | `v.capacity();` |
| `clear` | remove all elements | `v.clear();` |
| `sort` | ascending sort | `sort(v.begin(), v.end());` descending: `sort(v.rbegin(), v.rend());` or `sort(v.begin(), v.end(), greater<int>());` |
| `reverse` | reverse the range | `reverse(v.begin(), v.end());` |
| `accumulate` | sum | `accumulate(v.begin(), v.end(), 0LL);` use `0LL` or int overflows |
| `*min_element` / `*max_element` | smallest / largest value | `*max_element(v.begin(), v.end());` |
| `find` | iterator to value, `v.end()` if missing | `int i = find(v.begin(), v.end(), x) - v.begin();` |
| `count` | how many times x appears | `count(v.begin(), v.end(), x);` |
| `binary_search` | does x exist (sorted only) | `binary_search(v.begin(), v.end(), x);` |
| `rotate` | rotate left by k | `rotate(v.begin(), v.begin() + k, v.end());` |
| `unique` | move adjacent duplicates to the end (sort first) | `v.erase(unique(v.begin(), v.end()), v.end());` |
| `iota` | fill with 1,2,3,... | `iota(v.begin(), v.end(), 1);` |
| `swap` | swap two whole vectors, O(1) | `v.swap(v2);` |

Passing vectors to functions:

```cpp
void f(vector<int> &v);            // by reference: no copy, changes affect the original
void g(const vector<int> &v);      // by const reference: no copy, read only
void h(vector<int> v);             // by value: copies the whole vector, O(n)
```

## 4. Interview and important points

### Complexity

| Operation | Time |
|---|---|
| Access / update `v[i]` | O(1) |
| `push_back` | O(1) amortized |
| `pop_back` | O(1) |
| Insert / erase in the middle or front | O(n) |
| Search unsorted | O(n) |
| Search sorted (binary search) | O(log n) |
| Sort | O(n log n) |
| `clear` | O(n) |

### Commonly asked

1. **Vector vs array?** Array is fixed size (stack or global). Vector is dynamic, lives on the heap, grows automatically, and knows its own size.
2. **Size vs capacity?** Size is how many elements are in use. Capacity is how many it can hold before it has to re-allocate.
3. **Why is `push_back` O(1) amortized?** Most pushes are O(1). When capacity is full, it allocates a bigger block (about 2x) and copies everything, which is O(n). Spread over many pushes, the average is O(1).
4. **`reserve` vs `resize`?** `reserve(n)` only reserves space, size stays the same. `resize(n)` changes the size and creates elements. After `reserve(10)`, `v[0]` is still invalid.
5. **Does `clear()` free memory?** No. Size becomes 0, capacity stays. To really free it: `vector<int>().swap(v);` or `v.shrink_to_fit();`.
6. **Iterator invalidation:** `push_back` / `insert` can re-allocate, which invalidates all iterators, pointers and references to elements. `erase` invalidates the erased position and everything after it.
7. **Why is insert/erase at the front O(n)?** Every element after it must shift. Use `deque` if you need fast front operations.
8. **Why is `v.size() - 1` dangerous?** `size()` is unsigned. On an empty vector, `v.size() - 1` wraps around to a huge number. Cast with `(int)v.size() - 1`.
9. **Passing by value vs by reference?** By value copies the whole vector (O(n)). Pass by `&` or `const &`.
10. **Why does `vector<bool>` behave strangely?** It is a special bit-packed version. `v[i]` returns a proxy object, not a real `bool&`. Use `vector<char>` or `vector<int>` if that causes trouble.
11. **Out of bounds?** `v[i]` is undefined behavior, no error. `v.at(i)` throws an exception.
12. **How are vectors compared?** `==`, `<` etc. work elementwise (lexicographic), unlike C arrays.

### Mistakes to look out for

- `vector<int> v(n); v.push_back(x);` adds **after** n zeros. If you want an empty vector to push into, write `vector<int> v;` and optionally `v.reserve(n);`.
- Calling `pop_back()`, `back()` or `front()` on an empty vector is undefined behavior. Check `empty()` first.
- Erasing inside a range-for or index loop skips elements or breaks the loop. Loop backward, or use the erase-remove idiom.
- `remove` and `unique` do not shrink the vector. They move elements, and you must call `erase` to actually cut.
- `accumulate(v.begin(), v.end(), 0)` overflows for big sums, use `0LL`.
- `for (auto x : v)` copies each element. For big elements (strings, vectors) use `const auto &x`.
- Keeping a reference like `int &r = v[0];` and then calling `push_back` can leave `r` dangling.
- Valid indexes are `0` to `n-1`. `v.end()` points one past the last element, so ranges are `[begin, end)`.

### Common patterns

```cpp
// read n numbers
int n; cin >> n;
vector<int> v(n);
for (int &x : v) cin >> x;

// sum, max, min
long long sum = accumulate(v.begin(), v.end(), 0LL);
int mx = *max_element(v.begin(), v.end());

// prefix sum
vector<long long> pre(n + 1, 0);
for (int i = 0; i < n; i++) pre[i + 1] = pre[i] + v[i];
// sum of l..r = pre[r + 1] - pre[l]

// vector of pairs, sort by second
vector<pair<int,int>> p;
sort(p.begin(), p.end(), [](auto &a, auto &b) { return a.second < b.second; });

// sort + remove duplicates
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());

// adjacency list for a graph
vector<vector<int>> adj(n);
adj[u].push_back(w);
```

Classic problems built on vectors: two sum, subsets / permutations (backtracking), merge intervals, sliding window, kadane (max subarray), group anagrams, graph BFS / DFS with adjacency lists, DP tables.
