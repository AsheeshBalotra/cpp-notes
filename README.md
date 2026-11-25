# C++ notes for DSA interviews

Revision notes on C++ and the STL for data structures and algorithms interviews. Each file explains how a feature works in plain words, shows the syntax with examples and their output, and points out the bugs that come up in interview code.

## Files

| File                                         | Covers                                                                                                  |
|----------------------------------------------|---------------------------------------------------------------------------------------------------------|
| [`CplusNotes.md`](CplusNotes.md)             | 25 longer sections: C++ basics, iterators, binary search, trees, DP, sorting, heaps, BFS, graphs        |
| [`Notes_strings.md`](Notes_strings.md)       | ASCII and `char` arithmetic, `stoi`, `to_string`, `substr`, `find`, `erase`, case helpers, number bases |
| [`Notes_bits.md`](Notes_bits.md)             | bit operators, precedence, single-bit operations, tricks, XOR problems, bitmask subsets, binary         |
| [`Notes_vectors.md`](Notes_vectors.md)       | reverse iterators (`rbegin` / `rend`), `swap` with references vs copies, iterator invalidation          |
| [`Notes_maps.md`](Notes_maps.md)             | `unordered_map` and `map`: key checks, cost, key types, insert / erase, iterators, sorted keys          |
| [`Notes_containers.md`](Notes_containers.md) | `push` vs `insert`: which method each container has, costs, traps                                       |
| [`Notes_heap.md`](Notes_heap.md)             | `priority_queue`: how a heap works, complexity, syntax, top K, heap vs `set`                            |
| [`Notes_math.md`](Notes_math.md)             | modulo: negative remainders, `%` formulas, overflow and `long long` (`1LL`)                             |

## `CplusNotes.md` by topic

Many sections include the common bugs for that topic and how to fix them.

| Topic                          | Sections                                                                                                            |
|--------------------------------|---------------------------------------------------------------------------------------------------------------------|
| Compiling and language basics  | 1 compilation pipeline, 3 boolean truthiness, 6 pointers vs references, 7 the `&` operator, 16 char vs int          |
| Vectors, iterators and strings | 2 `[]` vs `.at()`, 4 `begin()` and `end()`, 5 iterator operations, 15 string reversal                               |
| Searching and loops            | 8 binary search, 14 majority element, 17 loop off-by-one, 18 slow and fast pointers                                 |
| Bits                           | 19 AND, OR, XOR, NOT and shifts: extra uses                                                                         |
| Recursion and DP               | 10 dynamic programming, 11 recursion direction                                                                      |
| Trees                          | 9 ancestors and lowest common ancestor, 12 tree recursion framework, 13 leaf vs null, 23 inorder and BST validation |
| Sorting and heaps              | 20 `lower_bound` and `upper_bound`, 21 sort comparators, 22 priority queue and heap                                 |
| Graphs                         | 24 BFS: mark visited at push time, 25 adjacency list vs matrix                                                      |

## Conventions

- Code assumes C++20 and `using namespace std;`.
- Problem numbers, such as 169 Majority Element, are LeetCode problem numbers.
- Warnings sit next to the operation they belong to, as **Trap:** bullets, with a section's main warning in a callout box. `Notes_bits.md` and `Notes_heap.md` collect theirs in a Traps section instead.
- Lines starting with `> TODO` mark topics still to be written up.
