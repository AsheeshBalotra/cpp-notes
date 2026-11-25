# Map Notes (DSA interviews)

Headers `<unordered_map>` and `<map>`. Sections 1, 3 and 4 work the same for both, section 2 compares them, and section 5 is `map` only.

## 1. Checking if a key exists

**`m[key]`:** returns the value stored for `key`. If the key is missing, it first **inserts** it with a default value (`0` for `int`) and returns that.

The default depends on the value type:
```
unordered_map<char, int>          -> 0
unordered_map<char, double>       -> 0.0
unordered_map<char, bool>         -> false
unordered_map<char, string>       -> ""  (empty string)
unordered_map<int, vector<int>>   -> {}  (empty vector, so m[k].push_back(x) works on a new key)
```

- `[]` returns a reference, so `m[key] = v` and `m[key]++` change the stored value.

So `if (m[key])` is a broken existence check, in two ways:
- **A real value of 0 counts as "not found",** because `0` is false. For example, a key stored at index 0.
- **A missing key gets inserted.** Every failed check quietly adds a fake entry with value 0. Later code treats it as real (`m[key]++` turns the fake 0 into 1), and the junk entries cost memory and time.

Both problems on one map:
```
m = {{5, 0}}            (key 5 stored at index 0)

if (m[5])     -> false     the key exists, but its value 0 reads as false
if (m[7])     -> false     and now m = {{5, 0}, {7, 0}}, a fake entry
m.size()      -> 2
```

Check without inserting:

| Call              | If the key is missing        | Use for                               |
|-------------------|------------------------------|---------------------------------------|
| `m[key]`          | inserts `key` with 0         | counting `m[x]++`, writing `m[x] = v` |
| `m.count(key)`    | returns 0, no insert         | does it exist?                        |
| `m.find(key)`     | returns `m.end()`, no insert | does it exist, then read the value    |
| `m.contains(key)` | returns false, no insert     | does it exist? (C++20)                |
| `m.at(key)`       | throws `out_of_range`        | reading a key you know exists         |

Check and read with one lookup:
```cpp
auto it = m.find(key);
if (it != m.end()) ans += it->second;      // it->first is the key, it->second the value
```

- `m[x]++` for counting is fine: a missing key starting at 0 is exactly what you want.

---

## 2. `map` vs `unordered_map`

**`unordered_map`:** a hash table. A hash function turns each key into the index of a bucket, so finding a key goes straight to its bucket. The keys are in no particular order.

**`map`:** a balanced binary search tree. Smaller keys go to the left and larger keys to the right, and the tree stays O(log n) levels tall. Iterating gives the keys in ascending order.

Cost of each operation:

| Operation                               | `unordered_map`               | `map`                       |
|-----------------------------------------|-------------------------------|-----------------------------|
| `[]`, `at`, `find`, `count`, `contains` | O(1) average, O(n) worst case | O(log n)                    |
| `insert`, `emplace`, `erase(key)`       | O(1) average, O(n) worst case | O(log n)                    |
| `size`, `empty`                         | O(1)                          | O(1)                        |
| iterate over every key, `clear`         | O(n)                          | O(n)                        |
| smallest / largest key                  | not available                 | O(1): `begin()`, `rbegin()` |
| `lower_bound`, `upper_bound`            | not available                 | O(log n)                    |

- **O(n) worst case:** if many keys land in the same bucket, a lookup checks them one by one. With normal inputs this is rare.
- `erase(it)` skips the search, so erasing by iterator is O(1) on average in both.
- Use `unordered_map` by default for counting and lookups. Use `map` when you need the keys in order, the smallest or largest key, or the nearest key to a value.
- `m.reserve(n)` (`unordered_map` only) makes room for `n` keys up front, so the table does not have to grow and move every key while you insert.

### Which key types work

`map` needs to compare two keys with `<`. `unordered_map` needs a hash function for the key, `std::hash<Key>`, and `==`.

| Key type                                 | `map`                              | `unordered_map`                  |
|------------------------------------------|------------------------------------|----------------------------------|
| `int`, `long long`, `char`, `string`     | works                              | works                            |
| `pair<int, int>`, `tuple`, `vector<int>` | works: compares element by element | does not compile: no `std::hash` |
| your own `struct`                        | needs `operator<`                  | needs a hash and `operator==`    |

Two ways to use a `pair` key in an `unordered_map`:
```cpp
// 1. Write the hash yourself
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        return hash<long long>()(((long long)p.first << 32) | (unsigned int)p.second);
    }
};
unordered_map<pair<int, int>, int, PairHash> seen;

// 2. Turn the pair into one number, for example a grid cell
long long key = (long long)r * cols + c;     // unique while 0 <= c < cols
unordered_map<long long, int> seen2;
```

- Without `PairHash`, `unordered_map<pair<int, int>, int>` fails with an error about `std::hash<std::pair<int, int>>`.
- If speed is not critical, `map<pair<int, int>, int>` works with no extra code, at O(log n) per operation.

---

## 3. Insert, update and erase

`m[key] = value` inserts the key or overwrites its value. `insert` and `emplace` never overwrite: if the key is already there, they change nothing.
```cpp
unordered_map<string, int> m;
m["a"] = 1;              // inserts:    {a:1}
m["a"] = 5;              // overwrites: {a:5}
m.insert({"b", 2});      // inserts:    {a:5, b:2}
m.insert({"b", 9});      // b is already there: nothing changes, b stays 2
m.emplace("c", 3);       // same as insert, with the key and value as separate arguments
```

`insert` and `emplace` return a pair: an iterator to the key, and `true` if they inserted it.
```
m.insert({"d", 4})     -> {iterator to d, true}     (inserted)
m.insert({"d", 7})     -> {iterator to d, false}    (already there, d stays 4)
```

- Read both parts with `auto [it, inserted] = m.insert({key, value});` (C++17).
- Because `insert` keeps the first value, `m.insert({key, i})` stores the first index where `key` appeared (523).

Three ways to erase:
```cpp
m.erase("a");            // by key: returns how many were removed, 1 (or 0 if "a" was missing)
m.erase(it);             // by iterator: returns an iterator to the next element
m.erase(first, last);    // a range [first, last): returns an iterator to the element after it
```

Size:
```cpp
m.size();                // number of keys
m.empty();               // true if there are no keys
m.clear();               // removes every key
```

- `size()` is the number of keys, so a key counts once however large its value is: after `m['a']++` twice and `m['b']++`, `m.size()` is 2.
- Setting a value to 0 does not remove the key: `m['a'] = 0` leaves `size()` unchanged. Only `erase` lowers it.
- Reading a missing key with `m[key]` inserts it and raises `size()` by 1 (section 1).

- **Trap:** `m[key]` does not compile on a `const` map (for example a `const map<int,int>&` parameter), because it may insert. Use `find` or `at`.
- **Trap:** `m.erase(m.find(key))` when `key` is missing erases `m.end()`, which is undefined. Erase by key (`m.erase(key)`), or check `!= m.end()` first.
- **Trap:** after `m.erase(it)`, `it` is invalid, so `++it` is undefined. Use the iterator that `erase` returns:
```cpp
for (auto it = m.begin(); it != m.end(); ) {
    if (it->second == 0) it = m.erase(it);   // erase gives back the next element
    else ++it;
}
```

- `set` and `unordered_set` have the same `insert`, `erase`, `find`, `count` and `contains`, with keys only: no `[]` and no `at`.

---

## 4. Iterators: reading and changing entries

**Map iterator:** points to one entry of the map, a `pair<const Key, Value>`. `find`, `begin`, `rbegin` and `lower_bound` return one (`insert` returns one inside its pair).

### Reading the key and the value

With `m = {a:1, b:2}` and `auto it = m.find("b");`:
```
it->first        -> "b"    (the key)
it->second       -> 2      (the value)
(*it).second     -> 2      (same as it->second)
```
```cpp
auto& [key, value] = *it;        // C++17: names for both. With &, value is the map's own value
```

### When to use `->` and when to use `.`

Look at what is on the left of the `->` or `.`:
- If it **points to** the thing you want (an iterator or a pointer), use `->`. `it->second` is the `second` of the entry that `it` points to.
- If it **is** the thing (the map, a `pair`, a node), use `.`. For example `m.size()`, `p.second`.

`it.first` does not compile: the iterator itself has no member called `first`, only the entry it points to has one. `*it` gives you that entry, so `(*it).first` works, and `it->first` is the short way to write it.

| On the left                                                  | Use                            | Example                          |
|--------------------------------------------------------------|--------------------------------|----------------------------------|
| an iterator, from `find`, `begin`, `rbegin` or `lower_bound` | `->`                           | `it->first`, `m.find(k)->second` |
| the entry itself: `*it`, or `p` in `for (auto& p : m)`       | `.`                            | `(*it).second`, `p.second`       |
| the map itself                                               | `.`                            | `m.find(k)`, `m.size()`          |
| the pair that `insert` returns                               | `.`, then `->` on its iterator | `m.insert({k, v}).first->second` |
| a node from `extract`                                        | `.`                            | `node.key()`, `node.mapped()`    |

- **Trap:** `*it.first` does not compile either. `.` binds tighter than `*`, so it means `*(it.first)`. Write `(*it).first`, or just `it->first`.
- **Trap:** the opposite mistake, `p->first` inside `for (auto& p : m)`, does not compile: `p` is the pair itself, so write `p.first`.
- Structured bindings need neither: after `auto& [key, value] = *it;`, `key` and `value` are plain names.
- The same rule holds for every iterator and for pointers. `root->left` uses `->` because `root` is a `TreeNode*`, and a `TreeNode` object would use `.` (pointers: section 6 of `CplusNotes.md`). A `vector<int>` iterator points to an `int`, which has no members, so you only write `*it`.

### What you can change through an iterator

| Iterator                                           | Key (`it->first`) | Value (`it->second`) |
|----------------------------------------------------|-------------------|----------------------|
| `iterator`, from a non-`const` map                 | no, read-only     | yes                  |
| `const_iterator`, from `cbegin()` or a `const` map | no                | no                   |

Through a normal iterator:
```cpp
it->second = 10;                 // OK: changes the value inside the map
it->second++;                    // OK
it->first = "z";                 // does not compile: the key is const
```

**Why the key is read-only:** the key decides where the entry lives: its place in `map`'s sorted tree, or its bucket in `unordered_map`. A key changed in place would sit in the wrong place, and lookups would miss it. The value plays no part in the position, so it can change freely.

To change a key, take the entry out and put it back under the new key:
```cpp
int v = it->second;              // copy the value first: erase(it) makes it invalid
m.erase(it);
m["z"] = v;
```

C++17 `extract` does the same without copying the value:
```cpp
auto node = m.extract(it);       // takes the entry out of the map
node.key() = "z";                // outside the map, the key can be changed
m.insert(std::move(node));       // puts it back under the new key
```

- A `set` element is read-only for the same reason: the whole element is the key, so `*it = 5` does not compile.

### Looping over every key and value

Two ways:
```cpp
for (auto& [key, value] : m)       // C++17: key is read-only, value can be changed
    value *= 2;

for (auto& p : m)                  // before C++17: p.first is the key, p.second the value
    cout << p.first << " " << p.second << "\n";
```

- `map` gives the keys in ascending order. `unordered_map` gives them in no particular order.
- **Trap:** `for (auto [key, value] : m)`, without `&`, copies each pair, so changing `value` does not change the map. For copies vs references, see section 2 of `Notes_vectors.md`.
- **Trap:** inside a loop over `m`, `m[newKey]` inserts a key. In `unordered_map` an insert can make the table grow and move every key, and then the loop's iterator is invalid, so continuing is undefined. In `map` the new key may be visited later in the same loop. Use `find` or `count` inside the loop (section 1).

---

## 5. `map` only: sorted keys

`unordered_map` has none of these, because its keys have no order. Examples on this map:
```cpp
map<int, string> m = {{10, "a"}, {20, "b"}, {30, "c"}};
```
```
m.begin()->first           -> 10         (smallest key)
m.rbegin()->first          -> 30         (largest key)
m.lower_bound(20)->first   -> 20         (first key >= 20)
m.upper_bound(20)->first   -> 30         (first key > 20)
m.lower_bound(25)->first   -> 30
m.lower_bound(35)          -> m.end()    (no key >= 35: do not dereference)
```

- What `lower_bound` and `upper_bound` mean, and why to check `end()`: section 20 of `CplusNotes.md`.
- `rbegin`, and removing the largest key with `m.erase(prev(m.end()))`: section 1 of `Notes_vectors.md`.
- `m.erase(m.begin(), m.lower_bound(x))` removes every key below `x`.

Largest key `<= x` (floor) and smallest key `>= x` (ceiling):
```cpp
auto ceilIt = m.lower_bound(x);                                 // smallest key >= x, or m.end()
auto after = m.upper_bound(x);                                  // first key > x
auto floorIt = (after == m.begin()) ? m.end() : prev(after);    // largest key <= x, or m.end()
```
```
keys {10, 20, 30}
x = 25   -> floor 20, ceiling 30
x = 30   -> floor 30, ceiling 30
x = 5    -> no floor, ceiling 10
x = 35   -> floor 30, no ceiling
```

**Practice:** 1 Two Sum, 380 Insert Delete GetRandom O(1), 523 Continuous Subarray Sum, 846 Hand of Straights, 981 Time Based Key-Value Store.
