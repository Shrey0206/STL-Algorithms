# Linked List (custom ListNode)

## 1. What is it

Linked list is a linear data structure where every element (**node**) stores a value and a **pointer to the next node**. Nodes are scattered in memory and connected only by pointers, so there is no index access. You reach a node only by walking from the **head**.

- Insert / delete at a known place is O(1) (just change pointers, nothing is shifted).
- Access by position is O(n), you have to walk.
- The last node points to `nullptr`. An empty list is `head == nullptr`.
- There is List STL container to use here but LeetCode and Codeforces use a hand-written node struct, so every operation is done with pointers.

| | Singly linked list | Doubly linked list | Circular linked list |
|---|---|---|---|
| Node has | `val`, `next` | `val`, `next`, `prev` | `val`, `next` (last points to head) |
| Walk | forward only | both directions | forward, in a loop |
| Delete a node when you only have that node | No (need the previous node) | Yes | No |
| Memory per node | less | more (extra pointer) | less |
| Used for | most interview problems | LRU cache, deque, browser history | round robin scheduling, Josephus |

| | `vector` / array | Linked list |
|---|---|---|
| Access `[i]` | O(1) | O(n) |
| Insert / erase at the front | O(n) | O(1) |
| Insert / erase in the middle (node known) | O(n) | O(1) |
| Memory | contiguous, cache friendly | scattered, extra pointer per node |
| Search | O(n), O(log n) if sorted | O(n) |

Rule of thumb: use `vector` by default. A linked list is used when the problem itself is about pointers (most interview problems), or when you need O(1) insert / delete at known positions.

Headers: none extra, `<bits/stdc++.h>` is enough.

In the snippets: `head` = pointer to the first node, `p`, `cur`, `prev` = node pointers, `x` = a value, `k` = a position (0 indexed), `v` = a `vector<int>`.

```cpp
// same as the one given by LeetCode
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

struct DNode {                          // doubly linked
    int val;
    DNode *prev;
    DNode *next;
    DNode(int x) : val(x), prev(nullptr), next(nullptr) {}
};
```

## 2. CRUD

### Create

```cpp
ListNode *head = nullptr;               // empty list
ListNode *head = new ListNode(1);       // one node
head->next = new ListNode(2);           // 1 -> 2
head->next->next = new ListNode(3);     // 1 -> 2 -> 3

ListNode *head = new ListNode(1, new ListNode(2, new ListNode(3)));   // same, in one line

```

### Insert

```cpp
// at the front, O(1)
ListNode* insertFront(ListNode *head, int x) {
    ListNode *newNode = new ListNode(x);
    newNode->next = head;
    return newNode;                     // the new head
}
head = insertFront(head, x);

// at the back, O(n) (O(1) if you keep a tail pointer)
ListNode* insertBack(ListNode *head, int x) {
    ListNode *newNode = new ListNode(x);
    if (!head) return newNode;
    ListNode *cur = head;
    while (cur->next) cur = cur->next;
    cur->next = newNode;
    return head;
}
head = insertBack(head, x);

// after a given node `cur`, O(1)
ListNode *newNode = new ListNode(x);
newNode->next = cur->next;              // first connect the new node to the rest
cur->next = newNode;                    // then connect cur to the new node (order matters)

// at position k (0 indexed), O(k)
ListNode* insertAt(ListNode *head, int k, int x) {
    if (k == 0) return insertFront(head, x);
    ListNode *cur = head;
    for (int i = 0; i < k - 1 && cur; i++) cur = cur->next;    // stop at node k-1
    if (!cur) return head;                                      // k is out of range
    ListNode *newNode = new ListNode(x);
    newNode->next = cur->next;
    cur->next = newNode;
    return head;
}
head = insertAt(head, k, x);

```

### Read

```cpp
// traverse
for (ListNode *cur = head; cur; cur = cur->next) cout << cur->val << ' ';

// length
int len(ListNode *head) {
    int n = 0;
    for (ListNode *cur = head; cur; cur = cur->next) n++;
    return n;
}

// k-th node (0 indexed), nullptr if out of range
ListNode* getAt(ListNode *head, int k) {
    while (head && k--) head = head->next;
    return head;
}

// find a value, nullptr if absent
ListNode* find(ListNode *head, int x) {
    while (head && head->val != x) head = head->next;
    return head;
}

// last node
ListNode *tail = head;
while (tail && tail->next) tail = tail->next;

// copy the values into a vector
vector<int> toVec(ListNode *head) {
    vector<int> v;
    for (; head; head = head->next) v.push_back(head->val);
    return v;
}

```

### Update

```cpp
head->val = 10;                         // change the value of head

// change every value
for (ListNode *cur = head; cur; cur = cur->next) cur->val *= 2;
```

### Delete

```cpp
// delete the first node, O(1)
if (head) {
    ListNode *curr = head;
    head = head->next;
    delete curr;
}

// delete the first node with value x (dummy node removes the special case for the head) 
ListNode* removeVal(ListNode *head, int x) {
    ListNode dummy(0);              // dummy precedes the real head
    dummy.next = head;
    ListNode *prev = &dummy;        // prev is the node before the one being checked
    while (prev->next) {
        if (prev->next->val == x) {
            ListNode *p = prev->next;   // node to delete
            prev->next = p->next;       // unlink it
            delete p;
            break;                      // remove this line to delete ALL nodes with value x
        } else prev = prev->next;
    }
    return dummy.next;              // new head
}

// delete at position k (0 indexed)
ListNode* deleteAt(ListNode *head, int k) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode *prev = &dummy;
    for (int i = 0; i < k && prev->next; i++) prev = prev->next;   // stop at node k-1
    if (prev->next) {
        ListNode *p = prev->next;
        prev->next = p->next;
        delete p;
    }
    return dummy.next;
}

// delete the last node
ListNode* deleteLast(ListNode *head) {
    if (!head || !head->next) { delete head; return nullptr; }
    ListNode *cur = head;
    while (cur->next->next) cur = cur->next;    // stop at the second last node
    delete cur->next;
    cur->next = nullptr;
    return head;
}

// free the whole list
while (head) {
    ListNode *p = head;
    head = head->next;
    delete p;
}

```

## 3. Important functions

There are no built in functions, these are the operations you write yourself:

| Operation | What it does | Time |
|---|---|---|
| insert at head | `head = insertFront(head, x);` | O(1) |
| insert after a node | `newNode->next = cur->next; cur->next = newNode;` | O(1) |
| insert at tail | walk to the last node (O(1) with a tail pointer) | O(n) |
| delete head | `head = head->next;` | O(1) |
| delete after a node | `cur->next = cur->next->next;` | O(1) |
| delete the last node | walk to the second last node | O(n) |
| search / get k-th | walk from head | O(n) |
| length | walk and count | O(n) |
| reverse | flip every `next` pointer | O(n) |
| find middle | slow and fast pointer | O(n) |
| merge two sorted lists | dummy node + tail | O(n + m) |
| sort | merge sort | O(n log n) |

Passing a list to a function:

```cpp
void f(ListNode *head);                 // pointer copy: you can change the nodes, but NOT where the caller's head points
void g(ListNode *&head);                // by reference: you can change the caller's head (insert / delete at the front)
ListNode* h(ListNode *head);            // or return the new head and assign it: head = h(head);
```

### Dummy node trick

Put a fake node before the head so inserting / deleting at the head is not a special case.

```cpp
ListNode dummy(0);
dummy.next = head;
ListNode *prev = &dummy;
// ... work with prev->next ...
return dummy.next;                      // the real head, even if the old head was deleted
```

### Slow and fast pointers

```cpp
ListNode *slow = head, *fast = head;
while (fast && fast->next) {
    slow = slow->next;                  // 1 step
    fast = fast->next->next;            // 2 steps
}
// slow is the middle (the second middle if the length is even)
```

## 4. Interview and important points

### Complexity

| Operation | Singly linked list | Doubly linked list | `vector` |
|---|---|---|---|
| Access `[i]` | O(n) | O(n) | O(1) |
| Insert / erase at the front | O(1) | O(1) | O(n) |
| Insert at the back | O(n), O(1) with a tail pointer | O(1) with a tail pointer | O(1) |
| Erase the back | O(n) | O(1) with a tail pointer | O(1) |
| Insert / erase after a known node | O(1) | O(1) | O(n) |
| Erase a node given only that node | not possible | O(1) | O(n) |
| Search for a value | O(n) | O(n) | O(n) |
| Length | O(n), O(1) if you store a size | O(n), O(1) if you store a size | O(1) |
| Reverse / merge two sorted | O(n) | O(n) | O(n) |
| Memory | n nodes + n pointers | n nodes + 2n pointers | n elements |

### Commonly asked

1. **What is a linked list?** A chain of nodes. Each node holds a value and a pointer to the next node.
2. **Array vs linked list?** Array: O(1) index, contiguous, cache friendly, O(n) insert / erase in the middle. Linked list: O(1) insert / erase when you hold the node, O(n) index, extra memory for pointers, scattered memory.
3. **Singly vs doubly linked list?** Doubly has a `prev` pointer too. You can go backward and delete a node in O(1) given just that node, but it uses more memory and more pointers to keep correct.
4. **Why is a linked list slow in practice for big data?** Nodes are scattered in memory, so the CPU cache misses a lot. A `vector` is usually faster even for inserts in the middle unless n is very large.
5. **How do you reverse a linked list?** Walk once and flip each `next` to point to the previous node, keeping `prev`, `cur`, `nxt`. O(n) time, O(1) space.
6. **How do you find the middle?** Slow and fast pointers. When fast reaches the end, slow is at the middle.
7. **How do you detect a cycle?** Floyd's algorithm: slow moves 1, fast moves 2. If they ever meet there is a cycle. If fast hits `nullptr` there is none. O(n) time, O(1) space.
8. **How do you find where the cycle starts?** After slow and fast meet, put one pointer back at the head. Move both 1 step at a time. They meet at the start of the cycle.
9. **How do you find the nth node from the end?** Move `fast` n steps ahead, then move `slow` and `fast` together. When `fast` reaches the end, `slow` is at the right place. With a dummy node you land on the node **before** the one to delete.
10. **How do you merge two sorted lists?** Keep a dummy node and a tail. Take the smaller head each time and attach it. Attach the leftover list at the end. O(n + m).
11. **How do you check if a list is a palindrome?** Find the middle, reverse the second half, compare both halves (and optionally reverse it back). O(n) time, O(1) space.
12. **How do you find the intersection of two lists?** Two pointers, one on each head. When a pointer reaches the end, move it to the head of the **other** list. They meet at the intersection (or both reach `nullptr`). O(n + m).
13. **How do you delete a node when you only have a pointer to that node?** Copy the next node's value into it and delete the next node: `node->val = node->next->val; node->next = node->next->next;`. It does not work for the last node.
14. **Why do you need a dummy node?** The head has no previous node, so inserting / deleting there is a special case. A dummy node before the head removes that special case.
15. **How do you sort a linked list?** Merge sort: split at the middle, sort both halves, merge. O(n log n) time. `std::sort` needs random access, so it does not work. Copying into a vector, sorting and writing back also works if extra memory is fine.
16. **How do you implement an LRU cache?** A doubly linked list (most recent at the front) plus a hash map from key to node. Get / put move the node to the front, eviction removes the tail. All operations O(1).
17. **How do you implement a stack and a queue with a linked list?** Stack: insert and remove at the head. Queue: insert at the tail (keep a tail pointer) and remove at the head.
18. **Where are linked lists used?** Stack / queue / deque implementations, LRU cache, hash table chaining, adjacency lists, undo / redo, browser history, music playlist, memory allocators.
19. **What is a circular linked list?** The last node points back to the head. You stop when you return to the start node, not at `nullptr`.
20. **How do you rotate a list by k places?** Find the length `n` and the tail, connect the tail to the head (make it circular), walk `n - k % n` steps from the head, cut the list there. The next node is the new head.

### Mistakes to look out for

- Dereferencing `nullptr`: `head->next` when `head` is `nullptr`, or `cur->next->next` without checking `cur->next`. Check before every `->`.
- Wrong order when inserting. `cur->next = p;` before `p->next = cur->next;` loses the rest of the list. Connect the new node to the rest first.
- Forgetting to update `head` when you insert / delete at the front. Return the new head, or take `ListNode *&head`.
- Losing a pointer while reversing. Save `nxt = cur->next` **before** you change `cur->next`.
- Walking too far: for insert / delete you need to stop at the node **before** the position (`k - 1`), not at the position.
- Memory leak: removing a node from the chain without `delete`. In interview problems it is usually fine to skip, in real code delete it.
- Using a node after `delete`. Save `p->next` before you delete `p`.
- Infinite loop on a list with a cycle. A simple `while (cur)` never ends. Detect the cycle first.
- Fast pointer: writing `while (fast->next && fast->next->next)` without checking `fast` first, or `fast->next->next` without checking `fast->next`. Use `while (fast && fast->next)`.
- Returning the address of a local dummy node's list. `dummy.next` is fine (it points to heap nodes), `&dummy` is not (it dies when the function ends).
- Not cutting the list after a split (`slow->next = nullptr`). Both halves still share the tail and the sort or merge loops forever.
- Doubly linked list: updating `next` but forgetting `prev` (or the other way round), and forgetting the head / tail cases.
- Comparing nodes by value when the problem wants the **same node** (intersection). Compare the pointers `a == b`.
- Not handling an empty list, one node, and two nodes. Test these three first.
- Counting the length many times inside a loop. That makes it O(n^2). Compute it once.

### Common patterns

```cpp
// reverse, O(n) time, O(1) space
ListNode* reverse(ListNode *head) {
    ListNode *prev = nullptr, *cur = head;
    while (cur) {
        ListNode *nxt = cur->next;      // save before breaking the link
        cur->next = prev;
        prev = cur;
        cur = nxt;
    }
    return prev;                        // the new head
}

// middle node (the second middle when the length is even)
ListNode* middle(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}

// detect a cycle (Floyd)
bool hasCycle(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}

// start of the cycle, nullptr if there is no cycle
ListNode* cycleStart(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            slow = head;
            while (slow != fast) {
                slow = slow->next;
                fast = fast->next;
            }
            return slow;
        }
    }
    return nullptr;
}

// merge two sorted lists
ListNode* mergeTwo(ListNode *a, ListNode *b) {
    ListNode dummy(0), *tail = &dummy;
    while (a && b) {
        if (a->val <= b->val) { tail->next = a; a = a->next; }
        else                  { tail->next = b; b = b->next; }
        tail = tail->next;
    }
    tail->next = a ? a : b;             // attach the leftover
    return dummy.next;
}

// remove the nth node from the end (n = 1 is the last node)
ListNode* removeNthFromEnd(ListNode *head, int n) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode *slow = &dummy, *fast = &dummy;
    while (n--) fast = fast->next;      // fast goes n steps ahead
    while (fast->next) {
        slow = slow->next;
        fast = fast->next;
    }
    ListNode *p = slow->next;           // slow is the node BEFORE the one to delete
    slow->next = p->next;
    delete p;
    return dummy.next;
}

// palindrome check
bool isPalindrome(ListNode *head) {
    ListNode *mid = middle(head);
    ListNode *second = reverse(mid);    // reverse the second half
    for (ListNode *a = head, *b = second; b; a = a->next, b = b->next)
        if (a->val != b->val) return false;
    return true;
}

// intersection of two lists (nullptr if none)
ListNode* intersection(ListNode *a, ListNode *b) {
    ListNode *p = a, *q = b;
    while (p != q) {
        p = p ? p->next : b;            // switch to the other list at the end
        q = q ? q->next : a;
    }
    return p;
}

// sort a list with merge sort, O(n log n)
ListNode* sortList(ListNode *head) {
    if (!head || !head->next) return head;
    ListNode *slow = head, *fast = head->next;  // first middle, so the split is even
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    ListNode *second = slow->next;
    slow->next = nullptr;               // cut the list into two
    return mergeTwo(sortList(head), sortList(second));
}

// delete a node when you only have that node (not the last one)
void deleteNode(ListNode *node) {
    ListNode *p = node->next;
    node->val = p->val;
    node->next = p->next;
    delete p;
}

// reorder list: L0 -> Ln -> L1 -> Ln-1 ...
void reorder(ListNode *head) {
    if (!head || !head->next) return;
    ListNode *mid = middle(head);
    ListNode *second = reverse(mid->next);      // reverse the second half
    mid->next = nullptr;                        // cut
    ListNode *first = head;
    while (second) {
        ListNode *a = first->next, *b = second->next;
        first->next = second;
        second->next = a;
        first = a;
        second = b;
    }
}

// print a list
void print(ListNode *head) {
    for (ListNode *cur = head; cur; cur = cur->next) cout << cur->val << (cur->next ? " -> " : "\n");
}
```

Classic problems built on linked lists: reverse linked list, merge two sorted lists, linked list cycle (I and II), middle of the linked list, remove nth node from end, palindrome linked list, intersection of two linked lists, reorder list, add two numbers, merge k sorted lists, sort list, copy list with random pointer, LRU cache, reverse nodes in k group, delete node in a linked list, rotate list.
