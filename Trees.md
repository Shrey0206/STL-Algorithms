# Trees (Binary Tree and BST, custom TreeNode)

## 1. What is it

Tree is a non linear, hierarchical data structure. It has one **root** node, and every node can have **children**. There are no cycles, and there is exactly one path between any two nodes. A **binary tree** is a tree where every node has **at most 2 children**: `left` and `right`.

- A node with no children is a **leaf**. An empty tree is `root == nullptr`.
- There is no STL container for trees: LeetCode and Codeforces give you (or expect) a hand-written node struct, so every operation is done with pointers and recursion.
- Most tree problems are solved by **recursion** (do the same thing for `left` and `right`) or by **BFS with a queue** (level by level).
- `set` and `map` are balanced BSTs inside, but you cannot reach their nodes. For interview problems you write the tree yourself.

| Term | Meaning |
|---|---|
| Root | the top node, has no parent |
| Parent / child | a node and the nodes directly below it |
| Leaf | a node with no children |
| Depth of a node | number of edges from the root to it (root has depth 0) |
| Height of a node | number of edges on the longest path down to a leaf (a leaf has height 0) |
| Height of the tree | height of the root. Many problems count **nodes** instead of edges, read the question |
| Level | all nodes with the same depth |
| Subtree | a node together with everything below it |

| Type | Rule |
|---|---|
| Full binary tree | every node has 0 or 2 children |
| Complete binary tree | all levels full except maybe the last, which is filled from the left (heap shape) |
| Perfect binary tree | all levels full, `2^h - 1` nodes for `h` levels |
| Balanced binary tree | for every node, the heights of the left and right subtrees differ by at most 1 |
| Skewed tree | every node has one child only, it is really a linked list |
| **BST** (binary search tree) | for every node: all values in the left subtree `<` node `<` all values in the right subtree |

| | Binary tree | BST |
|---|---|---|
| Order of values | none | left < node < right |
| Search | O(n) | O(h), O(log n) if balanced |
| Insert / delete | O(1) at a known place, O(n) to find it | O(h) |
| Inorder traversal gives | no useful order | **sorted** order |
| Duplicates | allowed | usually not allowed (or pick a side and be consistent) |

Rule of thumb: for a plain binary tree you use traversals (DFS / BFS). For a BST you use the ordering to go left or right and skip half the tree.

Headers: none extra, `<bits/stdc++.h>` is enough (`queue` for BFS, `stack` for iterative DFS).

In the snippets: `root` = pointer to the top node, `u`, `cur` = node pointers, `x` = a value, `h` = height, `res` = result vector, `v` = a `vector<int>`.

```cpp
// same as the one given by LeetCode
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};
```

A complete binary tree can also be stored in an array (0 indexed): children of `i` are `2i + 1` and `2i + 2`, parent is `(i - 1) / 2`. This is how a heap is stored.

## 2. CRUD

### Create

```cpp
TreeNode *root = nullptr;                       // empty tree
TreeNode *root = new TreeNode(1);               // one node
root->left = new TreeNode(2);                   //     1
root->right = new TreeNode(3);                  //    / \
root->left->left = new TreeNode(4);             //   2   3
                                                //  /
                                                // 4

// build from a level order vector, INT_MIN means null (LeetCode style: [1,2,3,null,4] = {1,2,3,INT_MIN,4})
TreeNode* build(const vector<int> &v) {
    if (v.empty() || v[0] == INT_MIN) return nullptr;
    TreeNode *root = new TreeNode(v[0]);
    queue<TreeNode*> q;
    q.push(root);
    int i = 1;
    while (!q.empty() && i < (int)v.size()) {
        TreeNode *cur = q.front();
        q.pop();
        if (i < (int)v.size() && v[i] != INT_MIN) { cur->left = new TreeNode(v[i]); q.push(cur->left); }
        i++;
        if (i < (int)v.size() && v[i] != INT_MIN) { cur->right = new TreeNode(v[i]); q.push(cur->right); }
        i++;
    }
    return root;
}
TreeNode *root = build({1, 2, 3, INT_MIN, 4});

// balanced BST from a sorted vector (take the middle as the root)
TreeNode* sortedToBST(const vector<int> &v, int l, int r) {
    if (l > r) return nullptr;
    int m = l + (r - l) / 2;
    return new TreeNode(v[m], sortedToBST(v, l, m - 1), sortedToBST(v, m + 1, r));
}
TreeNode *root = sortedToBST(v, 0, (int)v.size() - 1);
```

### Insert

```cpp
// BST: insert x, O(h), duplicates are ignored
TreeNode* insert(TreeNode *root, int x) {
    if (!root) return new TreeNode(x);
    if (x < root->val) root->left = insert(root->left, x);
    else if (x > root->val) root->right = insert(root->right, x);
    return root;
}
root = insert(root, x);

// BST: insert x, iterative
TreeNode* insertIter(TreeNode *root, int x) {
    TreeNode *newNode = new TreeNode(x);
    if (!root) return newNode;
    TreeNode *cur = root;
    while (true) {
        if (x < cur->val) {
            if (!cur->left) { cur->left = newNode; break; }
            cur = cur->left;
        } else {
            if (!cur->right) { cur->right = newNode; break; }
            cur = cur->right;
        }
    }
    return root;
}

// binary tree (no order): put x in the first empty place found level by level, O(n)
TreeNode* insertLevel(TreeNode *root, int x) {
    TreeNode *newNode = new TreeNode(x);
    if (!root) return newNode;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        TreeNode *cur = q.front();
        q.pop();
        if (!cur->left) { cur->left = newNode; break; }
        q.push(cur->left);
        if (!cur->right) { cur->right = newNode; break; }
        q.push(cur->right);
    }
    return root;
}
```

### Read

```cpp
// the 3 depth first traversals (recursive)
void preorder(TreeNode *u, vector<int> &res) {          // root, left, right
    if (!u) return;
    res.push_back(u->val);
    preorder(u->left, res);
    preorder(u->right, res);
}
void inorder(TreeNode *u, vector<int> &res) {           // left, root, right (sorted for a BST)
    if (!u) return;
    inorder(u->left, res);
    res.push_back(u->val);
    inorder(u->right, res);
}
void postorder(TreeNode *u, vector<int> &res) {         // left, right, root
    if (!u) return;
    postorder(u->left, res);
    postorder(u->right, res);
    res.push_back(u->val);
}

// level order (BFS), one vector per level
vector<vector<int>> levelOrder(TreeNode *root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();                              // nodes in the current level
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

// height (in nodes), 0 for an empty tree
int height(TreeNode *u) {
    if (!u) return 0;
    return 1 + max(height(u->left), height(u->right));
}

// number of nodes
int count(TreeNode *u) {
    if (!u) return 0;
    return 1 + count(u->left) + count(u->right);
}

// BST: search, O(h), nullptr if absent
TreeNode* search(TreeNode *root, int x) {
    while (root && root->val != x) root = x < root->val ? root->left : root->right;
    return root;
}

// BST: smallest and largest value
TreeNode* minNode(TreeNode *root) { while (root && root->left) root = root->left; return root; }
TreeNode* maxNode(TreeNode *root) { while (root && root->right) root = root->right; return root; }

// BST: inorder successor / predecessor values (the next bigger / smaller value than x), -1 if none
int successor(TreeNode *root, int x) {
    int ans = -1;
    while (root) {
        if (root->val > x) { ans = root->val; root = root->left; }
        else root = root->right;
    }
    return ans;
}
int predecessor(TreeNode *root, int x) {
    int ans = -1;
    while (root) {
        if (root->val < x) { ans = root->val; root = root->right; }
        else root = root->left;
    }
    return ans;
}
```

### Update

```cpp
root->val = 10;                                 // change the root value
root->left->val = 7;                            // change a node (make sure it exists)

// find a node and change it (plain binary tree, O(n))
void update(TreeNode *u, int oldVal, int newVal) {
    if (!u) return;
    if (u->val == oldVal) u->val = newVal;
    update(u->left, oldVal, newVal);
    update(u->right, oldVal, newVal);
}

// BST: changing a value in place can break the order.
// To update a value in a BST: erase the old one, then insert the new one.
root = erase(root, oldVal);
root = insert(root, newVal);
```

### Delete

```cpp
// BST: delete x, O(h)
TreeNode* erase(TreeNode *root, int x) {
    if (!root) return nullptr;
    if (x < root->val) root->left = erase(root->left, x);
    else if (x > root->val) root->right = erase(root->right, x);
    else {
        if (!root->left) {                      // 0 or 1 child (right): replace by the right child
            TreeNode *r = root->right;
            delete root;
            return r;
        }
        if (!root->right) {                     // 1 child (left): replace by the left child
            TreeNode *l = root->left;
            delete root;
            return l;
        }
        TreeNode *s = root->right;              // 2 children: find the inorder successor
        while (s->left) s = s->left;            // (smallest node in the right subtree)
        root->val = s->val;                     // copy its value here
        root->right = erase(root->right, s->val);   // and delete the successor
    }
    return root;
}
root = erase(root, x);

// free the whole tree (postorder: children first, then the node)
void freeTree(TreeNode *u) {
    if (!u) return;
    freeTree(u->left);
    freeTree(u->right);
    delete u;
}

// binary tree (no order): delete the first node with value x, O(n)
// no order to keep, so replace it with the deepest rightmost node (the last node in level order)
TreeNode* eraseLevel(TreeNode *root, int x) {
    if (!root) return nullptr;
    TreeNode *target = nullptr, *last = nullptr, *lastParent = nullptr;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        TreeNode *cur = q.front();
        q.pop();
        if (cur->val == x && !target) target = cur;                 // first match in level order
        if (cur->left)  { q.push(cur->left);  last = cur->left;  lastParent = cur; }
        if (cur->right) { q.push(cur->right); last = cur->right; lastParent = cur; }
    }
    if (!target) return root;                                       // x is not in the tree
    if (!last) { delete root; return nullptr; }                     // the tree had only the root
    target->val = last->val;                                        // copy the last node's value here
    if (lastParent->right == last) lastParent->right = nullptr;     // cut the last node off
    else lastParent->left = nullptr;
    delete last;
    return root;
}
root = eraseLevel(root, x);

// binary tree (no order): delete the whole subtree whose root has value x
TreeNode* eraseSubtree(TreeNode *root, int x) {
    if (!root) return nullptr;
    if (root->val == x) { freeTree(root); return nullptr; }         // the parent's pointer becomes nullptr
    root->left = eraseSubtree(root->left, x);
    root->right = eraseSubtree(root->right, x);
    return root;
}
root = eraseSubtree(root, x);
```

## 3. Important functions

There are no built in functions, these are the operations you write yourself:

| Operation | What it does | Time |
|---|---|---|
| preorder / inorder / postorder | depth first, visit root before / between / after its subtrees | O(n) |
| level order | BFS with a queue | O(n) |
| height / count nodes | recursion on both children | O(n) |
| BST search / insert / delete | go left or right by comparing | O(h) |
| binary tree delete | find the node, replace it with the deepest rightmost node | O(n) |
| BST min / max | go all the way left / right | O(h) |
| LCA | lowest common ancestor of two nodes | O(n), O(h) in a BST |
| validate BST | pass allowed range down (or check inorder is sorted) | O(n) |
| diameter | longest path between two nodes | O(n) |
| invert / mirror | swap `left` and `right` everywhere | O(n) |

The traversal order cheat sheet, for the tree `1` with children `2` (left) and `3` (right):

| Traversal | Order | Result |
|---|---|---|
| Preorder | root, left, right | 1 2 3 |
| Inorder | left, root, right | 2 1 3 |
| Postorder | left, right, root | 2 3 1 |
| Level order | level by level | 1 / 2 3 |

Passing a tree to a function:

```cpp
void f(TreeNode *root);                         // pointer copy: you can change the nodes, but NOT where the caller's root points
void g(TreeNode *&root);                        // by reference: you can change the caller's root
TreeNode* h(TreeNode *root);                    // or return the new root and assign it: root = h(root);
void k(TreeNode *u, vector<int> &res);          // collect results in a vector passed by reference
```

### Recursion pattern

Almost every tree problem has the same shape: handle `nullptr`, ask the left and right subtrees, combine their answers with the current node.

```cpp
int solve(TreeNode *u) {
    if (!u) return 0;                           // base case: empty tree
    int l = solve(u->left);                     // answer from the left subtree
    int r = solve(u->right);                    // answer from the right subtree
    return 1 + max(l, r);                       // combine (this one is the height)
}
```

### Iterative traversals (when recursion is too deep or the problem forbids it)

```cpp
vector<int> inorderIter(TreeNode *root) {
    vector<int> res;
    stack<TreeNode*> st;
    TreeNode *cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }      // go as far left as possible
        cur = st.top();
        st.pop();
        res.push_back(cur->val);
        cur = cur->right;
    }
    return res;
}

vector<int> preorderIter(TreeNode *root) {
    vector<int> res;
    if (!root) return res;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode *u = st.top();
        st.pop();
        res.push_back(u->val);
        if (u->right) st.push(u->right);                    // right first, so left is popped first
        if (u->left) st.push(u->left);
    }
    return res;
}

vector<int> postorderIter(TreeNode *root) {                 // reverse of (root, right, left)
    vector<int> res;
    if (!root) return res;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode *u = st.top();
        st.pop();
        res.push_back(u->val);
        if (u->left) st.push(u->left);
        if (u->right) st.push(u->right);
    }
    reverse(res.begin(), res.end());
    return res;
}
```

## 4. Interview and important points

### Complexity

| Operation | Binary tree | BST (balanced) | BST (skewed) |
|---|---|---|---|
| Search | O(n) | O(log n) | O(n) |
| Insert | O(n) to find the place (O(1) once you have it) | O(log n) | O(n) |
| Delete | O(n) | O(log n) | O(n) |
| Min / max | O(n) | O(log n) | O(n) |
| Any traversal | O(n) | O(n) | O(n) |
| Height | O(n) | O(n) | O(n) |
| Space (recursion stack) | O(h) | O(log n) | O(n) |
| Space (BFS queue) | O(w), w = max width, up to n / 2 | O(w) | O(1) |

`h` = height of the tree. For a balanced tree `h = log n`, for a skewed tree `h = n`.

### Commonly asked

1. **What is a binary tree?** A tree where every node has at most two children, `left` and `right`.
2. **What is a BST?** A binary tree where for every node, all values in its left subtree are smaller and all values in its right subtree are bigger.
3. **What does the inorder traversal of a BST give?** The values in **sorted** order. This is used to validate a BST and to find the kth smallest.
4. **What are the 4 traversals?** Preorder (root, left, right), inorder (left, root, right), postorder (left, right, root), all depth first. Level order is breadth first (queue).
5. **Which traversal for what?** Preorder: copy / serialize a tree. Inorder: sorted order in a BST. Postorder: delete a tree, compute heights (children before parent). Level order: shortest path, level wise answers.
6. **DFS vs BFS on a tree?** DFS (recursion or stack) uses O(h) memory and is shorter to write. BFS (queue) uses O(width) memory and goes level by level, so it is the one for "minimum depth" and "level order" problems.
7. **Height vs depth?** Depth is counted from the root down to the node. Height is counted from the node down to its deepest leaf. Check whether the problem counts edges or nodes.
8. **How do you find the height?** `1 + max(height(left), height(right))`, 0 for `nullptr`. O(n).
9. **How do you delete a node in a BST?** Three cases: no child (just remove it), one child (replace it with that child), two children (copy the inorder successor, the smallest value of the right subtree, into the node, then delete the successor).
10. **How do you validate a BST?** Pass an allowed range `(lo, hi)` down the recursion. Checking only `left < node < right` for each node is **wrong**, because a value deep in the left subtree can be bigger than an ancestor. Or check that the inorder is strictly increasing.
11. **How do you find the LCA?** Binary tree: recurse, if both sides return a node then the current node is the LCA, else return the non null side. BST: go left if both are smaller, right if both are bigger, else the current node is the LCA.
12. **What is a balanced tree and why does it matter?** Left and right heights differ by at most 1 at every node. It keeps the height at O(log n), so BST operations stay O(log n). A skewed BST degrades to O(n).
13. **What are self balancing trees?** AVL and Red-Black trees. They rotate nodes after insert / delete to stay balanced. `set`, `map`, `multiset` are Red-Black trees.
14. **What is the diameter of a tree?** The longest path (in edges) between any two nodes. At each node it is `height(left) + height(right)`, the answer is the maximum over all nodes.
15. **How do you serialize and deserialize a tree?** Preorder with a marker for `nullptr` (for example `#`), then rebuild by reading the same order. Level order also works.
16. **How do you build a tree from preorder and inorder?** The first element of preorder is the root. Find it in inorder: everything left of it is the left subtree, everything right is the right subtree. Recurse. Use a hash map from value to inorder index to make it O(n).
17. **Can you rebuild a tree from preorder and postorder only?** Only if it is a full binary tree (every node 0 or 2 children). Otherwise it is not unique.
18. **How many binary trees with n nodes?** The Catalan number `C(n) = (2n)! / ((n + 1)! n!)`. The same count applies to BSTs with n distinct keys.
19. **What is the maximum number of nodes at level `i` and in a tree of height `h`?** `2^i` at level `i` (root is level 0), and `2^(h+1) - 1` nodes in total (height counted in edges).
20. **How do you do Morris traversal?** Inorder traversal in O(1) extra space by temporarily linking the rightmost node of the left subtree back to the current node. It is rarely asked, know that it exists.
21. **Tree vs graph?** A tree is a connected graph with no cycles and `n - 1` edges. A graph can have cycles, many components and no root.
22. **Where are trees used?** File systems, HTML DOM, databases (B-trees for indexes), sorted sets and maps, heaps, expression parsing, tries, decision trees, networks routing.

### Mistakes to look out for

- Dereferencing `nullptr`: `u->left->val` without checking `u->left`. Check before every `->`, and put `if (!u) return` at the top of every recursive function.
- Forgetting the base case, which gives infinite recursion and a crash (stack overflow).
- Checking only `node->left->val < node->val` to validate a BST. You must pass the whole range down.
- Using `int` bounds when validating a BST (`INT_MIN` / `INT_MAX` as the range). Nodes can hold those values. Use `long long` bounds or `nullptr` bounds.
- Mixing up height counted in nodes and in edges. A single node has height 1 in nodes and 0 in edges.
- Searching a plain binary tree like a BST (going only one way). Without the order you must check both sides.
- Losing the root: not assigning back after insert / delete (`root = insert(root, x);`). The function returns the new root of the subtree.
- Deleting a node with two children and forgetting to delete the successor after copying its value, so the value now appears twice.
- Deep recursion on a skewed tree with 10^5 or 10^6 nodes can overflow the stack. Use an iterative version or BFS.
- Taking `q.size()` in the BFS inner loop condition while pushing children. Store it in `sz` before the loop.
- Pushing `nullptr` into the BFS queue and then reading `u->left` on it. Push only non null children.
- Forgetting to pass the result vector by reference (`vector<int> &res`) in a recursive traversal, so the caller sees nothing.
- Inserting duplicates into a BST without a rule. Decide: ignore them, or always send them to one side.
- Comparing nodes by value when the problem wants the **same node** (LCA, subtree). Compare the pointers `a == b`.
- Not testing: empty tree, one node, a skewed tree (only left or only right), and a perfect tree.

### Common patterns

```cpp
// maximum depth (in nodes)
int maxDepth(TreeNode *root) {
    if (!root) return 0;
    return 1 + max(maxDepth(root->left), maxDepth(root->right));
}

// minimum depth: nearest leaf (BFS stops at the first leaf)
int minDepth(TreeNode *root) {
    if (!root) return 0;
    queue<TreeNode*> q;
    q.push(root);
    int depth = 1;
    while (!q.empty()) {
        int sz = q.size();
        while (sz--) {
            TreeNode *u = q.front();
            q.pop();
            if (!u->left && !u->right) return depth;
            if (u->left) q.push(u->left);
            if (u->right) q.push(u->right);
        }
        depth++;
    }
    return depth;
}

// invert (mirror) a tree
TreeNode* invert(TreeNode *root) {
    if (!root) return nullptr;
    swap(root->left, root->right);
    invert(root->left);
    invert(root->right);
    return root;
}

// same tree
bool isSame(TreeNode *a, TreeNode *b) {
    if (!a && !b) return true;
    if (!a || !b) return false;
    return a->val == b->val && isSame(a->left, b->left) && isSame(a->right, b->right);
}

// symmetric tree (mirror of itself)
bool mirror(TreeNode *a, TreeNode *b) {
    if (!a && !b) return true;
    if (!a || !b) return false;
    return a->val == b->val && mirror(a->left, b->right) && mirror(a->right, b->left);
}
bool isSymmetric(TreeNode *root) { return !root || mirror(root->left, root->right); }

// balanced check: returns the height, or -1 if some subtree is unbalanced
int checkBalance(TreeNode *u) {
    if (!u) return 0;
    int l = checkBalance(u->left);
    if (l == -1) return -1;
    int r = checkBalance(u->right);
    if (r == -1 || abs(l - r) > 1) return -1;
    return 1 + max(l, r);
}
bool isBalanced(TreeNode *root) { return checkBalance(root) != -1; }

// diameter (in edges): longest path between two nodes
int diaDfs(TreeNode *u, int &best) {
    if (!u) return 0;
    int l = diaDfs(u->left, best);
    int r = diaDfs(u->right, best);
    best = max(best, l + r);                    // path that bends at u
    return 1 + max(l, r);
}
int diameter(TreeNode *root) {
    int best = 0;
    diaDfs(root, best);
    return best;
}

//LCA is the deepest node in a tree or directed acyclic graph that has both target nodes as descendants

// lowest common ancestor in a binary tree
TreeNode* lca(TreeNode *root, TreeNode *p, TreeNode *q) {
    if (!root || root == p || root == q) return root;
    TreeNode *l = lca(root->left, p, q);
    TreeNode *r = lca(root->right, p, q);
    if (l && r) return root;                    // p and q are on different sides
    return l ? l : r;
}

// lowest common ancestor in a BST
TreeNode* lcaBST(TreeNode *root, TreeNode *p, TreeNode *q) {
    while (root) {
        if (p->val < root->val && q->val < root->val) root = root->left;
        else if (p->val > root->val && q->val > root->val) root = root->right;
        else return root;
    }
    return nullptr;
}

// validate a BST (pass the allowed range down)
bool valid(TreeNode *u, long long lo, long long hi) {
    if (!u) return true;
    if (u->val <= lo || u->val >= hi) return false;
    return valid(u->left, lo, u->val) && valid(u->right, u->val, hi);
}
bool isValidBST(TreeNode *root) { return valid(root, LLONG_MIN, LLONG_MAX); }

// root to leaf path with a given sum
bool hasPathSum(TreeNode *root, int target) {
    if (!root) return false;
    if (!root->left && !root->right) return root->val == target;
    return hasPathSum(root->left, target - root->val) || hasPathSum(root->right, target - root->val);
}

// maximum path sum (path can start and end at any node)
int gain(TreeNode *u, int &best) {
    if (!u) return 0;
    int l = max(0, gain(u->left, best));        // ignore negative paths
    int r = max(0, gain(u->right, best));
    best = max(best, u->val + l + r);           // path that bends at u
    return u->val + max(l, r);
}
int maxPathSum(TreeNode *root) {
    int best = INT_MIN;
    gain(root, best);
    return best;
}

// right side view: the last node of every level
vector<int> rightSide(TreeNode *root) {
    vector<int> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            TreeNode *u = q.front();
            q.pop();
            if (i == sz - 1) res.push_back(u->val);
            if (u->left) q.push(u->left);
            if (u->right) q.push(u->right);
        }
    }
    return res;
}

// build a tree from preorder and inorder (distinct values)
TreeNode* go(const vector<int> &pre, int &i, int l, int r, unordered_map<int,int> &pos) {
    if (l > r) return nullptr;
    TreeNode *root = new TreeNode(pre[i++]);
    int m = pos[root->val];                     // root position in the inorder
    root->left = go(pre, i, l, m - 1, pos);
    root->right = go(pre, i, m + 1, r, pos);
    return root;
}
TreeNode* buildTree(const vector<int> &pre, const vector<int> &in) {
    unordered_map<int,int> pos;
    for (int i = 0; i < (int)in.size(); i++) pos[in[i]] = i;
    int i = 0;
    return go(pre, i, 0, (int)in.size() - 1, pos);
}

// all root to leaf paths as strings, e.g. "1->2->4"
void paths(TreeNode *u, string cur, vector<string> &res) {
    if (!u) return;
    cur += to_string(u->val);
    if (!u->left && !u->right) { res.push_back(cur); return; }
    cur += "->";
    paths(u->left, cur, res);
    paths(u->right, cur, res);
}

// print the tree level by level
void print(TreeNode *root) {
    for (auto &level : levelOrder(root)) {
        for (int x : level) cout << x << ' ';
        cout << '\n';
    }
}
```

Classic problems built on trees: maximum depth of binary tree, invert binary tree, diameter of binary tree, balanced binary tree, same tree, subtree of another tree, lowest common ancestor of a BST, lowest common ancestor of a binary tree, binary tree level order traversal, binary tree right side view, count good nodes, validate binary search tree, kth smallest element in a BST, construct binary tree from preorder and inorder traversal, binary tree maximum path sum, serialize and deserialize binary tree, symmetric tree, path sum, convert sorted array to BST.
