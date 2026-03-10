# Hashset (unordered_set) and Ordered Set (set)

## 1. What is it

A collection of **unique** values. There is no key → value pair like in a map, just the values. Adding a value that already exists does nothing. You use it to ask "is this value present?" very fast.

| | `unordered_set` (hashset) | `set` (ordered set) |
|---|---|---|
| Header | `<unordered_set>` | `<set>` |
| Inside | Hash table | Red-black tree (balanced BST) |
| Order of elements | No order | Always sorted |
| Insert / erase / find | O(1) average, O(n) worst | O(log n) |
| Element type must support | A hash function (value to bucket number) and an equality check (`a == b`) | Less-than check (`a < b`), to decide left or right in the tree |
| Pair / vector as element | Does **not** work by default (write a custom hash struct and pass it as the 2nd template argument, or use `set`) | Works |
| Extra powers | None, but faster | `lower_bound`, `upper_bound`, min / max element |

Rule of thumb: use `unordered_set` by default (duplicates, "have I seen this"). Switch to `set` only when you need sorted order or "next bigger / smaller element".

Same family: `multiset` (allows duplicates, sorted) and `unordered_multiset` (allows duplicates, no order).

Headers: `<unordered_set>`, `<set>`

In the snippets: `s` = `unordered_set<int>` (the baseline; the same syntax works for `set<int>` unless marked), `os` = `set<int>` (used only where sorted order matters), `x` = a value, `v`, `v1`, `v2` = vectors, `i`, `j` = indexes, `str` = a string.

## 2. CRUD

### Create

```cpp
unordered_set<int> s;                          // empty, no order
unordered_set<int> s = {1, 2, 3};              // list init
unordered_set<int> s(v.begin(), v.end());      // from a range, duplicates are dropped

set<int> os;                                   // ordered set: sorted ascending
set<int> os(v.begin(), v.end());               // sorted and unique in one line
set<int,greater<int>> desc;                    // sorted descending
```

### Insert

```cpp
s.insert(x);                       // adds x if absent, does nothing if present
s.emplace(x);                      // same, builds in place
s.insert({1, 2, 3});               // insert several
s.insert(v.begin(), v.end());      // insert a range

auto [it, added] = s.insert(x);    // added is true if x was new, it points to x either way
```

### Read

There is no `s[i]`. You cannot access by index, only check presence or loop.

```cpp
s.count(x);                        // 1 if present, 0 if not
s.contains(x);                     // C++20, true / false
s.find(x);                         // iterator, or s.end() if missing

if (s.find(x) != s.end()) cout << "found";

s.size();   s.empty();

for (int x : s) cout << x << ' ';  // order: random for unordered_set, sorted for set
for (auto it = s.begin(); it != s.end(); ++it) cout << *it;
```

### Update

Elements are `const`, so you cannot change a value in place. Erase the old one and insert the new one.

```cpp
s.erase(old);
s.insert(newVal);
```

### Delete

```cpp
s.erase(x);                        // remove by value, returns 1 if removed, 0 if absent
s.erase(it);                       // remove by iterator, returns the next iterator
s.erase(s.begin(), s.end());       // remove a range
s.clear();                         // remove everything

```

## 3. Important functions

| Function | What it does | Example |
|---|---|---|
| `insert` / `emplace` | add if absent | `s.insert(x);` |
| `erase` | remove by value or iterator | `s.erase(x);` |
| `find` | iterator to value, `end()` if missing | `if (s.find(x) != s.end())` |
| `count` / `contains` | does the value exist | `s.count(x)` |
| `size` / `empty` / `clear` | size, is-empty, remove all | `s.size();` |
| `swap` | swap two sets, O(1) | `s.swap(s2);` |
| `==` | same elements (order does not matter) | `s == s2` |

**Only for `set` (sorted), `os` = `set<int>`:**

| Function | What it does | Example |
|---|---|---|
| `*begin()` | smallest element | `*os.begin();` |
| `*rbegin()` | largest element | `*os.rbegin();` |
| `lower_bound(x)` | iterator to first element >= x | `auto it = os.lower_bound(x);` |
| `upper_bound(x)` | iterator to first element > x | `auto it = os.upper_bound(x);` |
| `prev(it)` / `next(it)` | neighbour iterators | `prev(os.lower_bound(x))` is the largest element < x (check `it != os.begin()` first) |
| `erase(begin())` | remove the smallest | `os.erase(os.begin());` |
| `erase(prev(end()))` | remove the largest | `os.erase(prev(os.end()));` |

**Set operations (sorted ranges, so use `set`):**

```cpp
set<int> a = {1, 2, 3}, b = {2, 3, 4}, r;     // use a fresh r for each call below
set_union(a.begin(), a.end(), b.begin(), b.end(), inserter(r, r.begin()));          // 1 2 3 4
set_intersection(a.begin(), a.end(), b.begin(), b.end(), inserter(r, r.begin()));   // 2 3
set_difference(a.begin(), a.end(), b.begin(), b.end(), inserter(r, r.begin()));     // 1  (in a, not in b)
includes(a.begin(), a.end(), b.begin(), b.end());                                   // does a contain all of b
```

Passing sets to functions:

```cpp
void f(unordered_set<int> &s);           // by reference: no copy
void g(const unordered_set<int> &s);     // read only
```

## 4. Interview and important points

### Complexity

| Operation | `unordered_set` | `set` |
|---|---|---|
| Insert / erase / find | O(1) avg, O(n) worst | O(log n) |
| `lower_bound` / `upper_bound` | not available | O(log n) |
| Iterate all | O(n), any order | O(n), sorted |
| Min / max element | O(n) | O(1) (`begin` / `rbegin`) |
| `size` | O(1) | O(1) |

### Commonly asked

1. **How does `unordered_set` work inside?** A hash table. The hash of the value picks a bucket. Collisions are handled by chaining (a list per bucket).
2. **How does `set` work inside?** A red-black tree (self-balancing BST). That is why elements are sorted and every operation is O(log n).
3. **Why is `unordered_set` O(n) in the worst case?** If many values land in the same bucket (bad hash or deliberately crafted input), that bucket becomes a long list.
4. **`set` vs `unordered_set`, when to use which?** `unordered_set` for plain "seen before?" checks and duplicates. `set` when you need sorted order, range queries or nearest value (`lower_bound`).
5. **`set` vs `map`?** A set stores only values. A map stores key and value. A set is basically a map with no value.
6. **Can you access `s[i]` or change an element?** No index access, and elements are `const`. Erase and insert instead.
7. **`set` vs `multiset`?** `multiset` allows duplicates. `ms.erase(x)` removes **all** copies, `ms.erase(ms.find(x))` removes only one. `count(x)` can be more than 1.
8. **How do you remove duplicates from a vector?** `set<int> t(v.begin(), v.end());` (also sorts), or `sort` + `unique` + `erase` (see the Vector notes), or `unordered_set` if order does not matter.
9. **Why can't `unordered_set` hold a `pair`?** There is no default hash for `pair`. Use `set<pair<int,int>>`, or write a custom hash.
10. **Iterator invalidation?** `set`: insert never invalidates iterators, erase only invalidates the erased one. `unordered_set`: a rehash (on insert) invalidates iterators, but not references to elements.
11. **Is the order in `unordered_set` stable?** No. It is unspecified and can change after a rehash.
12. **Find the k-th smallest in a `set`?** Not possible directly (no index). `next(os.begin(), k)` works but is O(k). A policy-based tree (`ordered_set` from GNU pbds) gives O(log n).

### Mistakes to look out for

- Erasing inside a range-for loop. `erase` invalidates the iterator, so use `it = s.erase(it)`.
- `std::lower_bound(os.begin(), os.end(), x)` is **O(n)** on a set, because set iterators cannot jump. Use the member `os.lower_bound(x)` instead (O(log n)).
- `distance(os.begin(), it)` is O(n), so a set cannot give "how many elements are smaller than x" quickly.
- Dereferencing `os.end()`, or `*os.begin()` / `*os.rbegin()` on an empty set, is undefined behavior. Check `empty()` first.
- `prev(os.begin())` is undefined behavior. Check `it != os.begin()` before using `prev`.
- `ms.erase(x)` on a `multiset` removes every copy, not just one.
- Anti-hash tests (Codeforces): crafted input makes `unordered_set` O(n) per operation. Use a custom hash (see the Hashmap notes) or switch to `set`.
- Expecting `unordered_set` to iterate in sorted order. It does not.

### Common patterns

```cpp
// contains duplicate
unordered_set<int> seen;
for (int x : v) {
    if (!seen.insert(x).second) return true;    // insert fails if x already there
}
return false;

// remove duplicates and sort
set<int> t(v.begin(), v.end());
vector<int> u(t.begin(), t.end());

// longest consecutive sequence, O(n)
unordered_set<int> st(v.begin(), v.end());
int best = 0;
for (int x : st) {
    if (st.count(x - 1)) continue;              // only start from the beginning of a run
    int len = 1;
    while (st.count(x + len)) len++;
    best = max(best, len);
}

// intersection of two arrays
unordered_set<int> a(v1.begin(), v1.end());
vector<int> res;
for (int x : unordered_set<int>(v2.begin(), v2.end()))
    if (a.count(x)) res.push_back(x);

// visited cells in a grid (pair needs set, or an encoded number)
set<pair<int,int>> vis;
vis.insert({i, j});
if (vis.count({i, j})) { /* already visited */ }

// nearest value >= x, and largest value < x (set)
auto it = os.lower_bound(x);
if (it != os.end()) cout << *it;                // smallest value >= x
if (it != os.begin()) cout << *prev(it);        // largest value < x

// sliding window: longest substring without repeating characters
unordered_set<char> w;
int l = 0, longest = 0;
for (int r = 0; r < (int)str.size(); r++) {
    while (w.count(str[r])) w.erase(str[l++]);
    w.insert(str[r]);
    longest = max(longest, r - l + 1);
}
```

Classic problems built on sets: contains duplicate, intersection / union of two arrays, longest consecutive sequence, longest substring without repeating characters, happy number (cycle detection), valid sudoku, find the missing / duplicate number, intervals and calendar problems (ordered set with `lower_bound`).
