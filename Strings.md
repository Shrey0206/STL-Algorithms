# String

## 1. What is it

A sequence of characters, like a `vector<char>` made for text. It grows and shrinks automatically, knows its own length, and is indexed from 0, so `s[0]` is the first character.

- **`string`** : the C++ one. Use this almost always. Needs `<string>`.
- **C-string** `char s[20]` : the old C way, an array of chars ending with `'\0'`. Fixed size, no helper functions, easy to overflow. Needs `<cstring>` for `strlen`, `strcpy` and so on.
- Text in double quotes `"abc"` is a C-string. Single quotes `'a'` is one `char`.

Headers: `<string>`, `<algorithm>` (reverse, sort, count...), `<cctype>` (isalpha, toupper...), `<sstream>` (stringstream)

In the snippets: `s`, `t` = `string`, `c` = a `char`, `i` = index, `pos` = starting index, `len` = length, `n` = a number.

## 2. CRUD

### Create

```cpp
string s;                          // empty
string s = "hello";                // from text
string s("hello");                 // same
string s(5, 'a');                  // "aaaaa"
string s(t);                       // copy of t (or s = t)
string s(t, pos, len);             // copy of len characters of t starting at pos
string s(t.begin(), t.end());      // from a range
string s = to_string(123);         // number to string: "123"
string s(1, c);                    // one char to a string
```

### Read

```cpp
s[i];                              // character at i, NO bounds check
s.at(i);                           // throws out_of_range if i is invalid
s.front();  s.back();              // first and last character
s.size();   s.length();            // same thing, number of characters
s.empty();                         // is it empty
s.c_str();                         // C-string (const char*), for printf and old C functions

for (char c : s) cout << c;        // loop over characters
for (char &c : s) c = toupper(c);  // & lets you change them
for (int i = 0; i < s.size(); i++) cout << s[i];

cin >> s;                          // reads one word (stops at space)
getline(cin, s);                   // reads the whole line, spaces included
```

### Update

```cpp
s[2] = 'x';                        // change one character
s += "abc";                        // append text (or s += c; for one char)
s.push_back(c);                    // append one char
s.append("abc");                   // append text
s.insert(pos, "abc");              // insert text at pos
s.insert(pos, 3, 'x');             // insert 'x' three times at pos
s.replace(pos, len, "abc");        // replace len characters starting at pos with "abc"
s.assign("new text");              // replace the whole content
s.resize(10, '*');                 // make size 10, new characters are '*' (or cut if smaller)
swap(s, t);                        // swap two strings
s = t;                             // copy
```

### Delete

```cpp
s.pop_back();                      // remove last character
s.erase(pos, len);                 // remove len characters starting at pos
s.erase(pos);                      // remove everything from pos to the end
s.erase(s.begin() + i);            // remove the character at index i
s.clear();                         // remove everything

// remove all occurrences of character c
s.erase(remove(s.begin(), s.end(), c), s.end());
```

## 3. Important functions

| Function | What it does | Example |
|---|---|---|
| `substr(pos, len)` | copy of a part, `len` is optional (default: to the end) | `s.substr(2, 3);` |
| `find(x)` | index of first match, `string::npos` if missing. `x` is a string or a char | `s.find("ab");` |
| `find(x, pos)` | same, but start searching from `pos` | `s.find('a', 3);` |
| `rfind(x)` | index of last match | `s.rfind('a');` |
| `find_first_of(set)` | first char that is any of `set` | `s.find_first_of("aeiou");` |
| `find_first_not_of(set)` | first char that is not in `set` | `s.find_first_not_of(' ');` |
| `compare(t)` | 0 if equal, negative if smaller, positive if bigger | `s.compare(t);` |
| `==`, `<`, `>` | compare content, dictionary order | `if (s == t)` |
| `+` | join two strings | `string r = s + t;` |
| `reverse` | reverse in place | `reverse(s.begin(), s.end());` |
| `sort` | sort the characters | `sort(s.begin(), s.end());` |
| `count` | how many times a char appears | `count(s.begin(), s.end(), 'a');` |
| `to_string(n)` | number to string | `to_string(42);` |
| `stoi` / `stoll` / `stod` | string to int / long long / double | `int n = stoi("123");` |
| `isalpha` / `isdigit` / `isalnum` | what kind of char | `isdigit(c)` |
| `islower` / `isupper` / `isspace` | what kind of char | `isupper(c)` |
| `tolower` / `toupper` | change case of one char | `c = tolower(c);` |
| `starts_with` / `ends_with` | prefix / suffix check (C++20) | `s.starts_with("ab");` |
| `stringstream` | split or parse text | see patterns below |

Searching:

```cpp
size_t p = s.find("ab");
if (p != string::npos) cout << "found at " << p;     // always compare with npos

// all occurrences
for (size_t p = s.find("ab"); p != string::npos; p = s.find("ab", p + 1))
    cout << p << ' ';
```

Case change on the whole string:

```cpp
transform(s.begin(), s.end(), s.begin(), [](unsigned char c) { return tolower(c); });
```

Passing strings to functions:

```cpp
void f(string &s);                 // by reference: no copy, changes affect the original
void g(const string &s);           // by const reference: no copy, read only
void h(string s);                  // by value: copies the whole string, O(n)
```

## 4. Interview and important points

### Complexity

| Operation | Time |
|---|---|
| Access / change `s[i]` | O(1) |
| `size`, `empty`, `push_back` (amortized), `pop_back` | O(1) |
| `s += t` | O(length of t), amortized |
| `s + t` (new string) | O(n + m) |
| `substr` | O(len), it makes a copy |
| `find` | O(n * m) worst case (n = text, m = pattern) |
| `==`, `<`, `compare` | O(n) |
| `insert` / `erase` in the middle or front | O(n) |
| `reverse` | O(n) |
| `sort` | O(n log n) |

### Commonly asked

1. **`string` vs `char[]`?** `string` grows automatically, knows its length, has many functions, and supports `=`, `+` and `==`. `char[]` is fixed size, ends with `'\0'`, and needs `strlen` / `strcpy` / `strcmp`.
2. **Are C++ strings mutable?** Yes. You can change characters in place (`s[i] = 'x'`). Java and Python strings are immutable.
3. **What is `string::npos`?** The "not found" value returned by `find`. It is the biggest possible `size_t`. Always compare with `string::npos`, not with `-1` or `0`.
4. **`length()` vs `size()`?** Same. Both are O(1).
5. **Why is `s.size() - 1` dangerous?** `size()` is unsigned. On an empty string, `s.size() - 1` wraps around to a huge number. Use `(int)s.size() - 1`.
6. **`"abc" + "def"` or `"abc" + 1`?** `"abc" + "def"` does not compile (two C-strings). `"abc" + 1` compiles but is pointer arithmetic, not concatenation. Make one side a `string`: `string("abc") + "def"`.
7. **Why is `s = s + c` in a loop slow?** It builds a brand new string every time, so the loop is O(n^2). `s += c` or `push_back` is amortized O(1).
8. **`+=` vs `append` vs `push_back`?** `+=` and `append` take a string or text, `push_back` takes one char. All append at the end.
9. **How do you compare strings?** `==`, `<`, `>` compare character by character (dictionary order). `"apple" < "banana"` is true, `"Z" < "a"` is true because uppercase has smaller ASCII values.
10. **`substr` cost?** It copies, so it is O(len). Using it inside a loop on a big string can make the solution O(n^2). Compare ranges with indexes when you can.
11. **How do you convert between char, digit and number?** Digit char to int: `c - '0'`. Int digit to char: `'0' + d`. Letter to index: `c - 'a'`. String to number: `stoi`, `stoll`, `stod`. Number to string: `to_string`.
12. **What happens with `s[i]` out of range?** Undefined behavior. `at(i)` throws. Note `s[s.size()]` is allowed and gives `'\0'`.
13. **What is SSO?** Small String Optimization. Short strings (about 15 characters or fewer in most compilers) are stored inside the string object itself, with no heap allocation.
14. **`string` vs `string_view`?** `string_view` (C++17) is a read-only window into existing text with no copy. Good for function parameters that only read.
15. **`cin >> s` vs `getline`?** `cin >>` reads one word and stops at whitespace. `getline` reads the whole line.

### Mistakes to look out for

- Mixing `cin >> n` and `getline(cin, s)`. The newline left behind makes `getline` read an empty line. Fix: `cin.ignore();` before the `getline`, or `cin >> ws;`.
- Comparing `find` with `-1`, or using the result as an index without checking `npos`.
- `s.erase(i)` erases from `i` to the **end**, not just one character. For one character use `s.erase(i, 1)`.
- Calling `pop_back()`, `back()` or `front()` on an empty string is undefined behavior. Check `empty()` first.
- `toupper(c)` / `tolower(c)` return `int`. Assigning back to a `char` is fine, but printing them directly (`cout << toupper(c)`) prints a number.
- Passing a negative `char` (non-ASCII text) to `isalpha`, `tolower` and friends is undefined behavior. Cast to `unsigned char` first.
- `stoi` throws `invalid_argument` if the text is not a number and `out_of_range` if it is too big.
- Erasing characters inside a range-for or an index loop skips elements. Use the erase-remove idiom, or loop backward.
- `to_string(3.0)` gives `"3.000000"`, not `"3"`. For control over decimals use `stringstream` or `printf`.
- `string s = 'a';` does not compile. Use `string s(1, 'a');` or `string s = "a";`.
- Strings are compared by content with `==`, but two `char*` are compared by address.

### Common patterns

```cpp
// frequency of lowercase letters
int cnt[26] = {};
for (char c : s) cnt[c - 'a']++;

// palindrome check (two pointers)
bool pal = true;
for (int l = 0, r = (int)s.size() - 1; l < r; l++, r--)
    if (s[l] != s[r]) { pal = false; break; }
// or: string r = s; reverse(r.begin(), r.end()); pal = (s == r);

// anagram check
string a = s, b = t;
sort(a.begin(), a.end());
sort(b.begin(), b.end());
bool anagram = (a == b);

// split by space
stringstream ss(s);
string w;
vector<string> words;
while (ss >> w) words.push_back(w);

// split by a delimiter (here a comma)
stringstream ss2(s);
while (getline(ss2, w, ',')) words.push_back(w);

// join words with a space
string joined;
for (int i = 0; i < (int)words.size(); i++) {
    if (i) joined += ' ';
    joined += words[i];
}

// reverse each word of a sentence
for (string &x : words) reverse(x.begin(), x.end());

// check if t is a substring of s
bool has = s.find(t) != string::npos;

// sliding window: longest substring without repeating characters
int last[256];
fill(last, last + 256, -1);
int l = 0, best = 0;
for (int r = 0; r < (int)s.size(); r++) {
    unsigned char c = s[r];
    if (last[c] >= l) l = last[c] + 1;
    last[c] = r;
    best = max(best, r - l + 1);
}

// count vowels
int vowels = 0;
for (char c : s) if (string("aeiouAEIOU").find(c) != string::npos) vowels++;
```

Classic problems built on strings: reverse a string, valid palindrome, valid anagram, longest substring without repeating characters, longest common prefix, group anagrams, string to integer (atoi), implement `strStr` (substring search), longest palindromic substring, count and say, reverse words in a sentence, string compression, KMP / Rabin-Karp for pattern search.
