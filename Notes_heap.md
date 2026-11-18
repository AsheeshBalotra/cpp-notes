# Heap / Priority Queue Notes (DSA interviews)

Header `<queue>`. Section 22 of `CplusNotes.md` (in this folder) adds heap vs priority queue as concepts, which containers can sit underneath, a method table against `queue` and `stack`, and when a plain vector beats a heap.

## 1. What a heap is

**Binary heap:** a tree where each node has at most 2 children (**binary**), and each parent is smaller than or equal to its children in a min-heap, or larger than or equal to them in a max-heap (the **heap rule**).

- The heap rule is the only rule. Siblings are not compared, so the tree is only partly sorted.
- Because of the rule, the smallest (min-heap) or largest (max-heap) value is always at the root, the top node.
- The tree is filled level by level, left to right, with no gaps.

**Priority queue:** a container that always gives you the smallest or largest item first. C++ `priority_queue` uses a binary heap to do this, and it is a max-heap by default.

**Why not a sorted vector:** inserting into a sorted vector shifts elements, which is O(n). A heap keeps only enough order to know the top, so push and pop are O(log n).

**Stored in a vector:** the tree is read level by level, left to right, into a `vector`, with no pointers. Index 0 is the root. Index arithmetic finds the parent and children:
```
parent(i) = (i - 1) / 2
left(i)   = 2 * i + 1
right(i)  = 2 * i + 2
```

Min-heap after pushing 5, 1, 9, 3:
```
vector:  [1, 3, 9, 5]

tree:        1          index 0
           /   \
          3     9       index 1, 2
         /
        5               index 3
```

**push (sift up):** add at the end, then swap with the parent while it beats the parent.
```
push 3 into [1, 5, 9]
[1, 5, 9, 3]   3 < parent 5, swap
[1, 3, 9, 5]   3 > parent 1, stop
```

**pop (sift down):** move the last element to the top, then swap it with the better child while that child beats it.
```
pop from [1, 3, 9, 5]
[5, 3, 9]      last element 5 moved to the top
[3, 5, 9]      smaller child 3 < 5, swap. No children left, stop
```

- Each swap moves one level, and the tree has log n levels, so push and pop are O(log n).

---

## 2. Time and space

| Operation                           | Time          |
|-------------------------------------|---------------|
| `top()`                             | O(1)          |
| `push(x)`                           | O(log n)      |
| `pop()`                             | O(log n)      |
| `size()`, `empty()`                 | O(1)          |
| Build from `n` values (constructor) | O(n)          |
| `n` separate `push` calls           | O(n log n)    |
| Find or remove any other element    | not supported |

Space is O(n).

- Push everything, then pop everything: that is heap sort, O(n log n).
- Build from a range in one call: `priority_queue<int> pq(v.begin(), v.end());`

### Why building from n values is O(n)

The constructor does not push the values one by one. It puts them all in the vector, then sifts down every parent node, from the last parent up to the root.

- A sift down costs at most the node's distance to the bottom of the tree.
- Half the nodes are leaves and never move. A quarter sit one level up and move at most once, an eighth move at most twice, and so on.
- Most nodes are near the bottom, so most sift downs are short.

Work per level, for n nodes:
```
level from the bottom   nodes   most swaps each
0 (leaves)              n/2     0
1                       n/4     1
2                       n/8     2
3                       n/16    3

total = n/4·1 + n/8·2 + n/16·3 + ...   which is less than n
```

- `n` separate `push` calls sift up instead. A sift up costs the distance to the top, and the many nodes near the bottom are the far ones, so the total is O(n log n).
- Measured with 65,536 values in ascending order: building took 131,040 comparisons (about 2n), and pushing one by one took 917,522 (about 14n).

---

## 3. Syntax and output

**Template arguments:** the types written inside `< >`. `priority_queue<T, Container, Compare>` takes three:
- `T`: the type of item stored.
- `Container`: where the items are kept. Always `vector<T>`.
- `Compare`: the **comparator**, a function that decides which of two items sits higher. The default is `less<T>`, which gives a max-heap.

Every type uses the same two forms:
```cpp
priority_queue<T> maxHeap;                           // largest on top
priority_queue<T, vector<T>, greater<T>> minHeap;    // smallest on top
```

- A min-heap needs all three arguments.

`int`:
```cpp
priority_queue<int> maxHeap;
priority_queue<int, vector<int>, greater<int>> minHeap;
```
```
push 5, 1, 9, 3
maxHeap.top()       -> 9
minHeap.top()       -> 1
maxHeap pop order   -> 9 5 3 1
minHeap pop order   -> 1 3 5 9
```

`pair<int,int>`, compared by `first`, then by `second` when `first` ties:
```cpp
priority_queue<pair<int,int>> maxHeap;
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> minHeap;

maxHeap.push({2, 10});           // or maxHeap.emplace(2, 10)
auto [a, b] = maxHeap.top();     // a = 2, b = 10
```
```
push {2,10}, {5,20}, {2,30}
maxHeap pop order   -> {5,20} {2,30} {2,10}
minHeap pop order   -> {2,10} {2,30} {5,20}
```

Shorter min-heap: `using` gives a type a short name.
```cpp
using P = pair<int,int>;
priority_queue<P, vector<P>, greater<P>> minHeap;
```

Other types use the same two forms. Only the order changes:

| Type                 | Push                        | Max-heap top | Min-heap top | Compared by                       |
|----------------------|-----------------------------|--------------|--------------|-----------------------------------|
| `string`             | `"bob", "al", "cy"`         | `"cy"`       | `"al"`       | dictionary order                  |
| `tuple<int,int,int>` | `{1,2,3}, {1,2,1}, {0,9,9}` | `{1,2,3}`    | `{0,9,9}`    | first, then second, then third    |
| `vector<int>`        | `{1,5}, {1,2,9}, {0,7}`     | `{1,5}`      | `{0,7}`      | element by element, like `string` |

- `long long`, `char` and `double` work like `int`.
- A `tuple` is handy for grid problems, such as `{dist, row, col}`. Read it with `auto [d, r, c] = pq.top();`.

Read, then remove:
```cpp
while (!pq.empty()) {
    int x = pq.top();     // read before pop()
    pq.pop();
}
```

**Custom order:** write a **lambda** (a small unnamed function written inline) that returns `true` when `a` should sit **below** `b`. So `>` gives a min-heap. That is the opposite of `sort`, where `>` gives descending order.
```cpp
auto cmp = [](const pair<int,int>& a, const pair<int,int>& b) { return a.second > b.second; };
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);   // decltype(cmp) = the lambda's type
```
```
push {1,30}, {2,10}, {3,20}
pop order           -> {2,10} {3,20} {1,30}     (smallest second first)
```

- Use the same form for your own `struct`.

---

## 4. Top K pattern

Keep the heap at size `k` by popping whenever it grows past `k`.
- k **largest** -> **min**-heap. The smallest of the k is on top, ready to be thrown out. Example: 703 Kth Largest Element in a Stream.
- k **smallest** -> **max**-heap. Example: 973 K Closest Points to Origin.

k largest with a min-heap:
```cpp
priority_queue<int, vector<int>, greater<int>> pq;    // k largest
for (int x : nums) {
    pq.push(x);
    if (pq.size() > k) pq.pop();
}
// pq.top() is the kth largest
```
```
nums = {3, 2, 1, 5, 6, 4}, k = 2   -> heap holds {5, 6}, top() = 5
```

- Time O(n log k), space O(k). Better than sorting, O(n log n), when k is small.

---

## 5. `priority_queue` vs `set`

|                    | `priority_queue`     | `set` / `multiset`                      |
|--------------------|----------------------|-----------------------------------------|
| Can see            | one end (`top`)      | both ends (`begin`, `rbegin`)           |
| Remove any element | no                   | O(log n)                                |
| Duplicates         | kept                 | `set` drops them, `multiset` keeps them |
| Iterate            | no                   | yes, in sorted order                    |
| Speed              | faster (flat vector) | slower (tree with pointers)             |

- Use a heap when you only need the top. Use a set when you need both ends or must delete specific elements.

---

## 6. Traps

- `top()` or `pop()` on an empty heap is undefined. Check `empty()` first.
- `pop()` returns nothing, so read `top()` first.
- `priority_queue<T, greater<T>>` does not compile: the second argument must be the container.
- You can only see `top()`. There are no iterators, so `for (int x : pq)` does not compile, and there is no `front()` (that is `queue`).

**Practice:** 703 Kth Largest Element in a Stream, 1046 Last Stone Weight, 215 Kth Largest Element in an Array, 347 Top K Frequent Elements, 973 K Closest Points to Origin, 295 Find Median from Data Stream.
