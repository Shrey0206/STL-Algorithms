# Stack

## 1. What is it

Stack is a linear data structure that follows **LIFO (Last In First Out)** Principle, the last element inserted is the first to be popped out. It means both insertion and deletion operations happen at one end only.

- You can only: add on **top**, look at the top, remove the top.
- No index access (`st[i]` does not exist), and no looping over it.
- It is a **container adapter**: it sits on top of another container (default `deque`) and only shows you the stack operations.
- Needs `<stack>`.

Headers: `<stack>`, `<vector>` (if you use a vector as a stack)

In the snippets: `st` = `stack<int>`, `sv` = `stack<vector<int>>` (a stack of vectors), `x` = a value.

## 2. CRUD

### Create

```cpp
stack<int> st;                          // empty
stack<int> st2(st);                     // copy of st (or st2 = st)
stack<pair<int,int>> sp;                // stack of pairs
stack<vector<int>> sv;                  // stack of vectors, each element is a whole vector
```

There is no list initialization (`stack<int> st = {1, 2, 3};` does not work). Push the values one by one.

### Insert (push)

```cpp
st.push(x);                             // put x on top
st.emplace(x);                          // same, builds the element in place (good for pairs / structs)
sp.emplace(1, 2);                       // no need to write make_pair

sv.push({1, 2, 3});                     // push a vector (list syntax works)
sv.push(vec);                           // push an existing vector (copies it)
sv.emplace(3, 0);                       // builds vector<int>(3, 0) = {0, 0, 0} in place
```

### Read

```cpp
st.top();                               // the top element, does NOT remove it
st.size();                              // number of elements
st.empty();                             // is it empty

if (!st.empty()) cout << st.top();      // always check before top()

sv.top();                               // the top vector
sv.top()[0];                            // first element of the top vector
sv.top().size();                        // size of the top vector
sv.top().back();                        // last element of the top vector
```

Only the top is visible. To see everything you have to pop (see below).

### Update

Only the top element can be changed.

```cpp
st.top() = 10;                          // overwrite the top
st.top()++;                             // change the top in place

sv.top()[0] = 9;                        // change one element inside the top vector
sv.top().push_back(4);                  // add to the top vector
sv.top().pop_back();                    // remove the last element of the top vector
// only the top vector can be reached, the vectors below it are not accessible
```

### Delete (pop)

```cpp
st.pop();                               // remove the top, returns NOTHING (void)

// read and remove together
int x = st.top();
st.pop();

// clear everything (stack has no clear())
while (!st.empty()) st.pop();
st = stack<int>();                      // or: replace with an empty stack

sv.pop();                               // removes the whole top vector, not one element
vector<int> v = sv.top();               // copy the top vector first if you still need it
sv.pop();
```

### Print all elements (empties it, so use a copy)

```cpp
stack<int> tmp = st;                    // copy, so st stays intact
while (!tmp.empty()) {
    cout << tmp.top() << ' ';           // prints from top to bottom
    tmp.pop();
}
```

## 3. Important functions

| Function | What it does | Example |
|---|---|---|
| `push(x)` | add on top | `st.push(5);` |
| `emplace(args)` | add on top, built in place | `sp.emplace(1, 2);` |
| `top()` | read / change the top | `int t = st.top();` |
| `pop()` | remove the top (returns nothing) | `st.pop();` |
| `size()` | number of elements | `st.size();` |
| `empty()` | true if no elements | `if (st.empty())` |
| `swap` | swap two stacks | `st.swap(st2);` |
| `==`, `<` | compare two stacks element by element | `if (st == st2)` |

Passing a stack to a function:

```cpp
void f(stack<int> &st);                 // by reference: no copy, changes affect the original
void g(stack<int> st);                  // by value: copies the whole stack, O(n)
```

### Using a vector as a stack (when you need to loop over it)

A `vector` can act as a stack and, unlike `stack`, you can loop over it and access any index.

```cpp
vector<int> st;
st.push_back(x);                        // push
st.back();                              // top
st.pop_back();                          // pop
st.empty();  st.size();
st.clear();                             // clear (stack has no clear)
for (int v : st) cout << v << ' ';      // print from bottom to top
```

## 4. Interview and important points

### Complexity

| Operation | Time |
|---|---|
| `push` / `emplace` | O(1) |
| `top` | O(1) |
| `pop` | O(1) |
| `size` / `empty` | O(1) |
| Search for a value | O(n), and you have to pop to do it |
| Copy | O(n) |

### Commonly asked

1. **What is a stack?** A LIFO structure. Insert and remove only at one end, the top.
2. **Why does `pop()` return nothing?** For exception safety. If copying the removed value threw an exception, the element would be lost. So you read with `top()` first and then call `pop()`.
3. **What happens if you call `top()` or `pop()` on an empty stack?** Undefined behavior. Always check `empty()` first.
4. **What does `stack` use inside?** A `deque` by default. You can choose `vector` or `list` as the second template argument.
5. **Why can't you iterate over a `stack`?** It is a container adapter that hides everything except the top. Use a `vector` if you need to loop.
6. **Stack vs queue?** Stack is LIFO (one end), queue is FIFO (add at the back, remove from the front).
7. **Where are stacks used?** Function calls (the call stack), undo / redo, browser back button, balancing brackets, evaluating expressions, DFS, backtracking.
8. **What is stack overflow?** Too much memory on the call stack, usually from deep or infinite recursion. It is about the system call stack, not the `stack` container.
9. **How do you implement a queue using two stacks?** Push into `in`. When you need to pop or peek and `out` is empty, move everything from `in` to `out`, which reverses the order. Pop from `out`. Amortized O(1).
10. **How do you implement a stack using queues?** After pushing `x` into a queue of size `n`, rotate the queue `n` times (move the front to the back) so `x` ends up at the front.
11. **What is a monotonic stack?** A stack that always stays sorted (increasing or decreasing). Before pushing, you pop everything that breaks the order. It solves "next greater / smaller element" problems in O(n).
12. **How do you get the minimum of a stack in O(1)?** Keep a second stack of minimums, or push pairs `(value, min so far)`.
13. **How do you convert recursion to iteration?** Use your own `stack` to hold what the recursion would hold on the call stack (for example iterative DFS or iterative tree traversal).

### Mistakes to look out for

- Calling `top()` or `pop()` on an empty stack. Check `!st.empty()`.
- Writing `int x = st.pop();`. It does not compile because `pop()` returns `void`. Use `top()` then `pop()`.
- Trying `st[i]`, a range-for, or `st.begin()`. They do not exist for `stack`.
- Looking for `st.clear()`. It does not exist. Use a `while` loop or assign an empty stack.
- Passing a `stack` by value into a recursive function. It copies the whole stack at each call.
- Holding `int &r = st.top();` and then calling `pop()`. The reference now points to a removed element.
- In the monotonic stack, forgetting to pop **while** the condition holds (using `if` instead of `while`).
- Leftover elements at the end. In bracket and monotonic problems, the elements still in the stack after the loop are often part of the answer (unmatched brackets, no next greater).

### Common patterns

```cpp
// balanced brackets: ()[]{}
bool valid(const string &s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') st.push(c);
        else {
            if (st.empty()) return false;
            char o = st.top(); st.pop();
            if ((c == ')' && o != '(') || (c == ']' && o != '[') || (c == '}' && o != '{')) return false;
        }
    }
    return st.empty();                          // leftover opening brackets = invalid
}

// next greater element to the right (monotonic stack), -1 if none
vector<int> nextGreater(const vector<int> &v) {
    int n = v.size();
    vector<int> res(n, -1);
    stack<int> st;                              // stores indexes, values decreasing
    for (int i = 0; i < n; i++) {
        while (!st.empty() && v[st.top()] < v[i]) {
            res[st.top()] = v[i];
            st.pop();
        }
        st.push(i);
    }
    return res;
}

// daily temperatures: days to wait for a warmer day
vector<int> wait(const vector<int> &t) {
    int n = t.size();
    vector<int> res(n, 0);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && t[st.top()] < t[i]) {
            res[st.top()] = i - st.top();
            st.pop();
        }
        st.push(i);
    }
    return res;
}

// min stack: push, pop and getMin all O(1)
stack<int> st, mn;
void push(int x) { st.push(x); if (mn.empty() || x <= mn.top()) mn.push(x); }
void pop()       { if (st.top() == mn.top()) mn.pop(); st.pop(); }
int getMin()     { return mn.top(); }

// evaluate postfix (reverse polish notation): {"2","1","+","3","*"} = 9
int evalRPN(const vector<string> &tokens) {
    stack<int> st;
    for (const string &t : tokens) {
        if (t == "+" || t == "-" || t == "*" || t == "/") {
            int b = st.top(); st.pop();         // right operand comes off first
            int a = st.top(); st.pop();
            if (t == "+") st.push(a + b);
            else if (t == "-") st.push(a - b);
            else if (t == "*") st.push(a * b);
            else st.push(a / b);
        } else st.push(stoi(t));
    }
    return st.top();
}

// reverse a string with a stack
string rev(const string &s) {
    stack<char> st;
    for (char c : s) st.push(c);
    string r;
    while (!st.empty()) { r += st.top(); st.pop(); }
    return r;
}

// iterative DFS on a graph
vector<bool> vis(n, false);
stack<int> dfs;
dfs.push(start);
while (!dfs.empty()) {
    int u = dfs.top(); dfs.pop();
    if (vis[u]) continue;
    vis[u] = true;
    for (int w : adj[u]) if (!vis[w]) dfs.push(w);
}
```

Classic problems built on stacks: valid parentheses, min stack, next greater element, daily temperatures, evaluate reverse polish notation, largest rectangle in histogram, trapping rain water, remove adjacent duplicates, decode string (`3[a]2[bc]`), implement queue using stacks, implement stack using queues, basic calculator, simplify path, asteroid collision, iterative tree traversals (inorder, preorder, postorder).
