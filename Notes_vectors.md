# Vector Notes (DSA interviews)

## 1. Reverse iterators: `rbegin` / `rend`

`rbegin()` points to the last element. `rend()` points one before the first element and works as a stop marker, like `end()`.

**`++it` on a reverse iterator moves backward**, from the last element toward the first.
```
v = {10, 20, 30, 40}

index:    -1     0     1     2     3     4
v:              10    20    30    40
          ^      ^                 ^     ^
        rend   begin            rbegin  end

normal  iterator, ++it:   begin  -> end     10 20 30 40
reverse iterator, ++it:   rbegin -> rend    40 30 20 10
```

Loop from the last element to the first:
```cpp
for (auto it = v.rbegin(); it != v.rend(); ++it)
    cout << *it << " ";                          // 40 30 20 10
```

Common uses:
```
*v.rbegin()                           -> 40                 (same as v.back())
*(v.rbegin() + 1)                     -> 30                 (one step back from the end)
vector<int> r(v.rbegin(), v.rend())   -> {40, 30, 20, 10}   (reversed copy)
string(s.rbegin(), s.rend())          -> "cba"              (s = "abc")
sort(v.rbegin(), v.rend())            -> {40, 30, 20, 10}   (sorts descending)
```

Take `k` elements from the back. The `it != rend()` check makes it safe when `k` is larger than the size.
```cpp
for (auto it = v.rbegin(); it != v.rend() && k > 0; ++it, --k)
    ans.push_back(*it);
```
```
v = {10, 20, 30, 40}, k = 2                   -> {40, 30}
s = set {{1,10}, {3,30}, {5,50}}, k = 2       -> {5,50} {3,30}    (the largest, since a set is sorted)
```

C++20 (header `<ranges>`):
```cpp
for (int x : v | views::reverse | views::take(k))   // 40 30
```

- Available on `vector`, `string`, `deque`, `list`, `set` and `map`. Not on `unordered_map` / `unordered_set`, because they have no order.
- A reverse iterator goes with `rend()`. Comparing it with `end()` does not compile.
- `it + k` works only on `vector`, `string` and `deque`. On `set` / `map`, use `next(it, k)`. Use `prev(s.end(), k)` instead of `s.end() - k`.
- `erase` needs a normal iterator. To remove the largest element of a set, use `s.erase(prev(s.end()))`, not `s.erase(s.rbegin())`.

---

## 2. `swap`: references vs copies

**Reference (`int&`):** another name for an existing variable. Changing it changes the original.

**`std::swap(a, b)` takes both arguments by reference,** so it swaps the actual objects you pass in.
```cpp
vector<int> v = {10, 20, 30};
swap(v[0], v[2]);            // v = {30, 20, 10}: v[i] returns a reference to the element
```

**Trap:** swapping with a local copy leaves the vector unchanged. Whether you get a copy depends on how you declare the variable:
- `int last = v.back();` makes a **copy**, so changing `last` does not touch the vector.
- `int& last = v.back();` makes a **reference** to the vector's last slot, so changing `last` changes the vector. Be careful: after `pop_back()`, that reference points to a removed element, so do not use it.

> TODO: Expand — besides `pop_back`, which vector operations make an existing reference or iterator invalid (for example `push_back` when the vector grows, `insert`, `erase`).

Swapping with a copy vs a reference, starting from `v = {10, 20, 30}`:
```
int  last = v.back(); swap(v[0], last)   -> v = {30, 20, 30}   (last = 10, v[2] unchanged)
int& last = v.back(); swap(v[0], last)   -> v = {30, 20, 10}   (last is v[2] itself)
```

- To swap two elements of a container, pass the slots themselves: `swap(v[i], v[j])`. `v.front()` and `v.back()` also return references.
- `swap(a, 5)` does not compile: a reference needs a real variable, not a literal.
- `swap(v1, v2)` on two whole vectors is O(1): it exchanges their internal buffers.

**Practice:** 380 Insert Delete GetRandom O(1).
