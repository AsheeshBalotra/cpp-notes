# Containers Notes (DSA interviews)

Which method adds, removes and reads an element in each STL container, and why the names differ. Related: `Notes_maps.md` (`insert` vs `m[k]`), `Notes_heap.md` (`priority_queue`), `Notes_vectors.md` (iterators).

## 1. `push` or `insert`?

**Container adaptor:** a wrapper that limits an underlying container to a few operations. `stack`, `queue` and `priority_queue` are adaptors. By default `stack` and `queue` sit on a `deque`, and `priority_queue` sits on a `vector`.

**The rule:** `push` is for containers where you choose the end, and `insert` is for containers where the container (or an iterator) decides the position.
- **`push` / `pop`:** the adaptors. You can only put an element in and take one out, and the container decides which one comes out (the last, the first, or the largest).
- **`push_back` / `push_front`:** `deque` and `list`, where you choose the end. `vector` has only `push_back`.
- **`insert(it, x)`:** `vector`, `deque` and `list`, where you choose the position with an iterator.
- **`insert(x)`:** `set` and `map` (and their `unordered_` versions). The container places the element by sorted order or by hash, so you give only the value.

What each container has:

| Container              | Add                                        | Remove                               | Look at                   |
|------------------------|--------------------------------------------|--------------------------------------|---------------------------|
| `stack`                | `push`                                     | `pop`                                | `top`                     |
| `queue`                | `push` (to the back)                       | `pop` (from the front)               | `front`, `back`           |
| `priority_queue`       | `push`                                     | `pop`                                | `top`                     |
| `vector`               | `push_back`, `insert(it, x)`               | `pop_back`, `erase(it)`              | `[]`, `front`, `back`     |
| `deque`                | `push_back`, `push_front`, `insert(it, x)` | `pop_back`, `pop_front`, `erase(it)` | `[]`, `front`, `back`     |
| `list`                 | `push_back`, `push_front`, `insert(it, x)` | `pop_back`, `pop_front`, `erase(it)` | `front`, `back` (no `[]`) |
| `set`, `unordered_set` | `insert(x)`                                | `erase(x)`                           | `find`, `count`           |
| `map`, `unordered_map` | `insert({k, v})`, `m[k] = v`               | `erase(k)`                           | `find`, `m[k]`            |

One use of each:
```cpp
stack<int> st;           st.push(1); st.push(2);                 // st.top()   -> 2
queue<int> q;            q.push(1);  q.push(2);                  // q.front()  -> 1
priority_queue<int> pq;  pq.push(1); pq.push(2);                 // pq.top()   -> 2
deque<int> d;            d.push_front(1); d.push_back(2);        // d          -> 1 2
list<int> l = {1, 3};    l.insert(next(l.begin()), 2);           // l          -> 1 2 3
set<int> s;              s.insert(5);
```

`emplace`, `emplace_back` and `emplace_front` follow the same position rules as `insert`, `push_back` and `push_front`. They build the element in place from its constructor arguments, so you skip a copy.

### Cost

```
push / pop on stack, queue          O(1)
push / pop on priority_queue        O(log n)
push_back / pop_back                O(1)    (vector: amortized)
insert / erase in the middle        O(n) on vector and deque, O(1) on list if you already hold the iterator
set / map insert                    O(log n)
unordered_set / unordered_map       O(1) average
```

- Finding the position in a `list` is still O(n), so a middle insert is O(1) only when you already have the iterator.
- `vector` has no `push_front`, because it would shift every element. Use a `deque`.

### Traps

- **Trap:** the wrong method name does not compile. Each of these gives `no member named ... in ...`:
```
priority_queue<int> p; p.insert(1)      -> no member named 'insert'
stack<int> p;          p.insert(1)      -> no member named 'insert'
vector<int> p;         p.push_front(1)  -> no member named 'push_front'
set<int> p;            p.push_back(1)   -> no member named 'push_back'
queue<int> p;          p.push_front(1)  -> no member named 'push_front'
```
- **Trap:** `top()`, `front()` and `back()` on an empty container are undefined behaviour. Check `empty()` first. On an empty `stack`, the crash shows up inside `stl_deque.h`, because the `stack` sits on a `deque`.
- **Trap:** `pop()` returns nothing for `stack`, `queue` and `priority_queue`. Read the value with `top()` or `front()`, then call `pop()`.
- **Which element comes out:** `stack` gives the last pushed (`top`), `queue` gives the first pushed (`front`), and `priority_queue` gives the largest (`top`), or the smallest with `greater`.

**Practice:** 20 Valid Parentheses, 155 Min Stack, 739 Daily Temperatures, 347 Top K Frequent Elements, 253 Meeting Rooms II.
