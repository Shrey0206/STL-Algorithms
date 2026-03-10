# Hashmap (unordered_map) and Ordered Map (map)

## 1. What is it

Both store **key → value** pairs. Every key is unique, and you look up a value by its key, not by an index.

| | `unordered_map` (hashmap) | `map` (ordered map) |
|---|---|---|
| Header | `<unordered_map>` | `<map>` |
| Inside | Hash table | Red-black tree (balanced BST) |
| Order of keys | No order | Always sorted |
| Lookup / insert / erase | O(1) average, O(n) worst | O(log n) |
| Key type must support | A hash function (key to bucket number) and an equality check (`a == b`) | Less-than check (`a < b`), to decide left or right in the tree |
| Pair / vector as key | Does **not** work by default (write a custom hash struct and pass it as the 3rd template argument, or use `map`) | Works |
| Extra powers | None, but faster | `lower_bound`, `upper_bound`, min / max key |

Rule of thumb: use `unordered_map` by default (lookup, counting). Switch to `map` only when you need sorted keys or "next bigger / smaller key".

In the snippets: `m` = `unordered_map<int,int>` (the baseline; the same syntax works for `map<int,int>` unless marked), `om` = `map<int,int>` (used only where sorted order matters), `k` = a key, `val` = a value.

## 2. CRUD

### Create

```cpp
unordered_map<int,int> m;                      // empty, no order
unordered_map<string,int> ms = {{"a", 1}, {"b", 2}};   // list init
unordered_map<int,int> c(m.begin(), m.end());  // from a range
m.reserve(1000);                               // pre-size the table, avoids rehashing

map<int,int> om;                               // ordered map: sorted by key
map<int,int,greater<int>> desc;                // keys in descending order
```

### Insert

```cpp
m[k] = val;                        // insert, or overwrite if k exists
m.insert({k, val});                // insert only if k is NOT present (does not overwrite)
m.emplace(k, val);                 // same as insert, builds in place
m.insert_or_assign(k, val);        // C++17: insert or overwrite
m.try_emplace(k, val);             // C++17: insert only if absent, does not build val if present

m[k]++;                            // counting: missing key starts at 0
m[k].push_back(x);                 // unordered_map<int, vector<int>>: missing key starts as empty vector
```

### Read

```cpp
m[k];                              // value of k. If k is missing, it INSERTS k with default value (0)
m.at(k);                           // value of k, throws out_of_range if missing
m.count(k);                        // 1 if present, 0 if not
m.contains(k);                     // C++20, true / false
m.find(k);                         // iterator, or m.end() if missing

auto it = m.find(k);
if (it != m.end()) cout << it->first << ' ' << it->second;   // key and value

m.size();   m.empty();

for (auto &[key, v] : m) cout << key << ' ' << v;   // C++17 structured binding (order: random for unordered_map, sorted for map)
for (auto &p : m) cout << p.first << ' ' << p.second;   // older style
```

### Update

```cpp
m[k] = val;                        // overwrite
m[k] += 5;                         // change in place
m.at(k) = val;                     // overwrite, throws if k missing
m.find(k)->second = val;           // via iterator (make sure k exists)
// keys cannot be changed. To change a key: erase it and insert the new one.
```

### Delete

```cpp
m.erase(k);                        // remove by key, returns 1 if removed, 0 if absent
m.erase(it);                       // remove by iterator, returns the next iterator
m.erase(m.begin(), m.end());       // remove a range
m.clear();                         // remove everything

// erase while looping (erase invalidates `it`, so use its return value)
for (auto it = m.begin(); it != m.end(); ) {
    if (it->second == 0) it = m.erase(it);
    else ++it;
}
```

## 3. Important functions

| Function | What it does | Example |
|---|---|---|
| `operator[]` | get / set value, inserts if missing | `m[k]++;` |
| `at` | get value, throws if missing | `m.at(k);` |
| `insert` / `emplace` | add only if key is absent | `m.insert({k, val});` |
| `insert_or_assign` | add or overwrite | `m.insert_or_assign(k, val);` |
| `find` | iterator to key, `end()` if missing | `if (m.find(k) != m.end())` |
| `count` / `contains` | does the key exist | `m.count(k)` |
| `erase` | remove by key or iterator | `m.erase(k);` |
| `size` / `empty` / `clear` | size, is-empty, remove all | `m.size();` |
| `swap` | swap two maps, O(1) | `m.swap(m2);` |

**Only for `map` (sorted), `om` = `map<int,int>`:**

| Function | What it does | Example |
|---|---|---|
| `begin()->first` | smallest key | `om.begin()->first;` |
| `rbegin()->first` | largest key | `om.rbegin()->first;` |
| `lower_bound(k)` | iterator to first key >= k | `auto it = om.lower_bound(k);` |
| `upper_bound(k)` | iterator to first key > k | `auto it = om.upper_bound(k);` |
| `prev(it)` / `next(it)` | neighbour iterators | `prev(om.lower_bound(k))` is the largest key < k (check `it != om.begin()` first) |
| `erase(m.begin())` | remove the smallest key | `om.erase(om.begin());` |
| `erase(prev(m.end()))` | remove the largest key | `om.erase(prev(om.end()));` |



Passing maps to functions:

```cpp
void f(unordered_map<int,int> &m);           // by reference: no copy
void g(const unordered_map<int,int> &m);     // read only (can't use m[k] here, use find / at)
```

## 4. Interview and important points

### Complexity

| Operation | `unordered_map` | `map` |
|---|---|---|
| Insert / erase / find / `[]` | O(1) avg, O(n) worst | O(log n) |
| `lower_bound` / `upper_bound` | not available | O(log n) |
| Iterate all | O(n), any order | O(n), sorted |
| Min / max key | O(n) | O(1) (`begin` / `rbegin`) |

### Commonly asked

1. **How does `unordered_map` work inside?** A hash table. The hash of the key picks a bucket. Collisions are handled by chaining (a list per bucket).
2. **How does `map` work inside?** A red-black tree (self-balancing BST). That is why keys are sorted and every operation is O(log n).
3. **Why is `unordered_map` O(n) in the worst case?** If many keys land in the same bucket (bad hash or deliberately crafted keys), that bucket becomes a long list.
4. **`map` vs `unordered_map`, when to use which?** `unordered_map` for plain lookup / counting. `map` when you need sorted order, range queries or nearest key (`lower_bound`).
5. **What does `m[k]` do for a missing key?** It inserts `k` with a default value and returns it. So `if (m[k])` or `m[k] == 0` as an existence check **grows the map**. Use `count`, `find` or `contains`.
6. **`m[k] = v` vs `insert({k, v})`?** `[]` overwrites. `insert` does nothing if the key exists.
7. **`insert` vs `emplace` vs `try_emplace`?** `insert` takes a ready pair. `emplace` builds the pair in place. `try_emplace` does not even build the value if the key exists.
8. **Why can't `unordered_map` use `pair` as a key?** There is no default hash for `pair`. Use `map<pair<int,int>, T>`, or write a custom hash.
9. **Iterator invalidation?** `map`: insert never invalidates iterators, erase only invalidates the erased one. `unordered_map`: a rehash (on insert) invalidates iterators, but not references to elements.
10. **Can you change a key?** No, keys are const. Erase and insert again.
11. **Is the order in `unordered_map` stable?** No. It is unspecified and can change after a rehash.
12. **`map` vs `multimap`?** `multimap` allows duplicate keys (no `[]`). Same idea for `unordered_multimap`.
13. **What if the key type is a custom struct?** `map` needs `operator<` (or a comparator). `unordered_map` needs a hash function and `operator==`.
14. **Why is `unordered_map` often slower than expected?** Hashing cost, cache misses (nodes scattered in memory) and rehashing. `reserve` helps. For small key ranges a plain array is faster.

### Mistakes to look out for

- Using `m[k]` just to check if a key exists. It silently inserts the key.
- Erasing inside a range-for loop. `erase` invalidates the iterator, so use `it = m.erase(it)`.
- Calling `m[k]` on a `const map`. It does not compile. Use `at()` or `find`.
- `prev(om.begin())` or dereferencing `om.end()` is undefined behavior. Check before using `lower_bound` results.
- `om.rbegin()` / `om.begin()` on an empty map is undefined behavior. Check `empty()` first.
- Anti-hash tests (Codeforces): crafted keys make `unordered_map` O(n) per operation. Use a custom hash or switch to `map`.
- Overflow: values counted into `int` can overflow, use `long long` as the value type when counts get big.
- Expecting `unordered_map` to iterate in sorted order. It does not.

### Common patterns

```cpp
// frequency count
unordered_map<int,int> freq;
for (int x : v) freq[x]++;

// two sum (find a pair adding to target)
unordered_map<int,int> seen;                   // value -> index
for (int i = 0; i < n; i++) {
    if (seen.count(target - v[i])) return {seen[target - v[i]], i};
    seen[v[i]] = i;
}

// group anagrams
unordered_map<string, vector<string>> groups;
for (string s : words) {
    string key = s;
    sort(key.begin(), key.end());
    groups[key].push_back(s);
}

// most frequent element
int best = 0, ans = 0;
for (auto &[key, c] : freq) if (c > best) best = c, ans = key;

// nearest key >= x, and largest key < x (map)
auto it = om.lower_bound(x);
if (it != om.end()) cout << it->first;         // smallest key >= x
if (it != om.begin()) cout << prev(it)->first; // largest key < x

// pair as key
map<pair<int,int>, int> grid;
grid[{2, 3}] = 5;

// custom hash for unordered_map (safe against anti-hash tests)
struct Hash {
    size_t operator()(long long x) const {
        x += chrono::steady_clock::now().time_since_epoch().count();
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
        x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
        return x ^ (x >> 31);
    }
};
unordered_map<long long, int, Hash> safe;
```

Classic problems built on maps: two sum, group anagrams, longest substring without repeating characters, subarray sum equals k (prefix sum + map), top K frequent elements, LRU cache (map + list), first non-repeating character, intervals / calendar problems (ordered map with `lower_bound`).
s