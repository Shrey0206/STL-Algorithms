# Array

## 1. What is it

A fixed-size collection of data items of the same data type stored in contiguous (sequential) memory locations. Every box has an index starting from 0, so you can jump to any box instantly.

- **C array** `int a[5]` : the basic one, no header needed.
- **std::array** `array<int,5>` : same thing but safer. Knows its size, can be copied and compared. Needs `<array>`.

Headers: `<algorithm>` (sort, find...), `<numeric>` (accumulate, iota), `<cstring>` (memset), `<climits>` (INT_MAX)

In the snippets: `a` = C array, `x` = std::array, `n` = how many elements you are actually using.

## 2. CRUD

### Create

```cpp
int a[5];                  // garbage values if local, zeros if global
int a[5] = {};             // all zeros
int a[5] = {1, 2};         // 1 2 0 0 0
int a[] = {1, 2, 3};       // size becomes 3
int m[3][4] = {};          // 2D array, zeros

array<int,5> x = {};       // std::array, zeros
array<int,3> y = {1,2,3};
array w{1,2,3};            // C++17, type deduced
```

### Read

```cpp
a[i];  x[i];               // access by index, NO bounds check
x.at(i);                   // same but throws error if i is out of range
x.front();  x.back();      // first and last (std::array only)
a[n-1];                    // last element for a C array

for (int v : a) cout << v;         // loop over values
for (int &v : a) v *= 2;           // & lets you change them
for (int i = 0; i < n; i++) cout << a[i];
```

### Update

```cpp
a[2] = 10;                 // set one element
swap(a[0], a[1]);          // swap two elements
x = y;                     // copy whole std::array (C array cannot do this)
```

### Delete

Array size is fixed, so there is no real delete. You shift elements and keep track of `n` yourself.

```cpp
// delete element at index i
copy(a + i + 1, a + n, a + i);  // copy(from_begin, from_end_exclusive, to_begin)
n--;

// clear
n = 0;
```

## 3. Important functions

Pattern: most take `(begin, end)`. For a C array that is `(a, a + n)`. End is **exclusive**.

| Function | What it does | Example |
|---|---|---|
| `sort` | ascending sort | `sort(a, a + n);` descending: `sort(a, a + n, greater<int>());` |
| `reverse` | reverse the range | `reverse(a, a + n);` |
| `fill` | set all to a value | `fill(a, a + n, 0);` |
| `memset` | fast fill, only for `0` or `-1` | `memset(a, 0, sizeof a);` |
| `iota` | fill with 1,2,3,... | `iota(a, a + n, 1);` |
| `accumulate` | sum | `accumulate(a, a + n, 0LL);` use `0LL` or int overflows |
| `*min_element` / `*max_element` | smallest / largest value in array | `*max_element(a, a + n);` |
| `max_element(...) - a` | index of the largest | `int idx = max_element(a, a + n) - a;` |
| `find` | position of a value, `n` if missing | `int i = find(a, a + n, v) - a;` |
| `count` | how many times v appears | `count(a, a + n, v);` |
| `binary_search` | does v exist (sorted only) | `binary_search(a, a + n, v);` |
| `lower_bound` | first index with value >= v (sorted) | `lower_bound(a, a + n, v) - a;` |
| `upper_bound` | first index with value > v (sorted) | `upper_bound(a, a + n, v) - a;` |
| `rotate` | rotate left by k | `rotate(a, a + k, a + n);` |
| `unique` | remove adjacent duplicates (sort first) | `n = unique(a, a + n) - a;` |
| `size` | length (C++17) | `size(a)` |
| `copy` | copy a range | `copy(a, a + n, b);` |

Passing arrays to functions:

```cpp
void f(int a[], int n);        // becomes a pointer, so you must pass the size
void g(array<int,5> &x);       // std::array keeps its size, pass by reference
```

## 4. 2D Array

An array of arrays. Think of a grid: `m[row][col]`. Stored row by row in memory.

**Which to use:** size known / fixed limits (N ≤ 1000) → C array or `array<array<>>`, usually global. Size comes from input or rows differ in length → `vector<vector<int>>`.

### Create

```cpp
int m[3][4] = {};                              // 3 rows, 4 cols, zeros
int m[3][4] = {{1,2,3,4}, {5,6,7,8}, {9,10,11,12}};
int dp[1005][1005];                            // global: zeros

array<array<int,4>,3> g = {};                  // std::array version

vector<vector<int>> v(n, vector<int>(m, 0));   // n rows, m cols, zeros
```

### Read

```cpp
m[i][j];                                       // row i, col j
v[i][j];

for (int i = 0; i < 3; i++)
    for (int j = 0; j < 4; j++)
        cout << m[i][j] << ' ';

for (auto &row : m)                            // range-for (use & so rows are not copied)
    for (int val : row) cout << val << ' ';

v.size();                                      // rows
v[0].size();                                   // cols
```

### Update

```cpp
m[1][2] = 10;
memset(m, 0, sizeof m);                        // only 0 or -1
fill(&m[0][0], &m[0][0] + 12, 7);              // fill whole C 2D array (3*4 = 12)
for (auto &row : v) fill(row.begin(), row.end(), 7);   // fill a 2D vector
```

### Delete

```cpp
// C array: no real delete, just overwrite or ignore
// vector:
v.erase(v.begin() + i);                        // remove row i
v[i].erase(v[i].begin() + j);                  // remove element j from row i
v.clear();                                     // remove everything
```

### Passing to functions

```cpp
void f(int m[][4], int rows);                  // only the first dimension can be omitted
void g(vector<vector<int>> &v);                // pass by reference, else it copies everything
```

### Useful 2D operations

```cpp
// sort each row
for (auto &row : v) sort(row.begin(), row.end());

// sort rows by first column
sort(v.begin(), v.end());                      // compares row by row (lexicographic)

// transpose (R x C -> C x R)
vector<vector<int>> t(C, vector<int>(R));
for (int i = 0; i < R; i++)
    for (int j = 0; j < C; j++) t[j][i] = v[i][j];
```

### 2D points to remember

- `v(n, m)` is wrong. Write `v(n, vector<int>(m))`.
- Rows of a vector can have different lengths (jagged). A C 2D array cannot.
- C 2D array is one contiguous block. A 2D vector is separate blocks per row, so it is a bit slower.
- A big local 2D array overflows the stack. Make it global or use vector.
- Row-major traversal (`i` outer, `j` inner) is faster because of cache. Looping column by column is slower.
- `sizeof m / sizeof m[0]` gives rows, `sizeof m[0] / sizeof m[0][0]` gives cols (only where `m` is a real array).

## 5. Interview and important points

### Complexity

| Operation | Time |
|---|---|
| Access / update `a[i]` | O(1) |
| Insert / delete in the middle | O(n) |
| Search unsorted | O(n) |
| Search sorted (binary search) | O(log n) |
| Sort | O(n log n) |

### Commonly asked

1. **Array vs vector?** Array is fixed size (stack or global). Vector is dynamic and lives on the heap, grows automatically.
2. **Why is access O(1)?** Elements are contiguous, so address = start + i * sizeof(type).
3. **Why is insert/delete O(n)?** Everything after that position must shift.
4. **Array decays to a pointer** when passed to a function. `sizeof a` inside the function gives the pointer size, not the array size. That is why you pass `n` separately.
5. **`a` vs `&a[0]`** : the array name is the address of the first element. `a + i` is the address of the i-th element, `*(a + i)` is same as `a[i]`.
6. **What if you go out of bounds?** Undefined behavior. No error, it may crash or silently give garbage. `at()` throws an exception instead.
7. **Local vs global array:** local is uninitialized (garbage), global is zeros.
8. **Why is a big local array dangerous?** The stack is small (about 1 to 8 MB). Something like `int a[10000000]` inside `main` crashes. Make it global or use vector.
9. **Can you compare C arrays with `==`?** It compares addresses, not contents. Use `equal()` or `std::array`.
10. **Can you assign `b = a` for C arrays?** No. Use `copy` or `memcpy`. `std::array` allows it.
11. **Variable length array** `int a[n]` with runtime `n` is not standard C++ (GCC allows it). Use vector or `new int[n]`.

### Mistakes to look out for

- `sort(a, a + n)` : end is exclusive, do not write `a + n - 1`.
- `accumulate(a, a + n, 0)` overflows for big sums, use `0LL`.
- `memset(a, 1, sizeof a)` does **not** make every element 1, it sets every byte to 1. Only use 0, -1 (or `0x3f`, which makes each int about 1e9, handy as infinity).
- Using `INT_MAX` as infinity then adding to it overflows. Use `1e9` or `INT_MAX / 2`.
- Looping backward with `size_t` (`for (size_t i = n - 1; i >= 0; i--)`) never ends, because unsigned never goes below 0. Use `int`.
- Off by one: valid indexes are `0` to `n-1`.

### Common patterns

```cpp
// frequency array for lowercase letters
int cnt[26] = {};
for (char c : s) cnt[c - 'a']++;

// grid movement (up, down, left, right)
int dx[] = {0, 0, 1, -1}, dy[] = {1, -1, 0, 0};
for (int k = 0; k < 4; k++) {
    int nx = i + dx[k], ny = j + dy[k];
}

// anagram check with std::array (== works)
array<int,26> f1{}, f2{};
bool anagram = (f1 == f2);
```

