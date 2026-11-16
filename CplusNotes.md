# C++ Refresher Notes (DSA interviews)

**Abbreviations**
- **LCA:** lowest common ancestor
- **UB:** undefined behavior

Longer C++ and DSA notes, one section per topic: language basics, iterators, binary search, trees, DP, slow and fast pointers, bits, sorting, heaps, BFS and graphs. Where a section overlaps a `Notes_*.md` file in this folder, it points there and keeps only what that file does not have.

## 1. Compilation pipeline (`a.cpp` -> `a`)

1. **Preprocessing** (`g++ -E`): expands `#include`, `#define` and `#ifdef`, and strips comments. Output: plain C++ text (`.i`).
2. **Compilation** (`g++ -S`): parses, type-checks, optimizes, and lowers the code to human-readable assembly (`.s`).
3. **Assembly** (`g++ -c`): converts the assembly to binary machine code, packaged as an object file with unresolved symbols (`.o`).
4. **Linking** (`g++ a.o -o a`): resolves symbols against libraries (`libstdc++`, `libc`) and merges everything into one runnable binary.
5. **Running** (`./a`): the OS loader maps the binary into memory and jumps to the entry point. The CPU runs native machine code directly, with no runtime translation.

---

## 2. `vector`: `operator[]` vs `.at()`

- `pop_back()` only decrements the size. It does not clear or overwrite the memory, so the old value is physically still there at that address.
- `v[i]` does **no** bounds checking. Reading past `size()` is undefined behavior (UB), and it may "accidentally" show stale data instead of failing.
- `v.at(i)` **does** bounds checking: it throws `std::out_of_range` if `i >= size()`, so it is safer for catching real bugs.
- **Lesson:** never trust `v[i]` past `size() - 1`, even if it prints something plausible. Use `.at()` when unsure.

---

## 3. Boolean truthiness

- `0` (all bits 0) is false. **Any** non-zero value (1, -1, 42, ...) is true, not just 1.
- This applies to ints, pointers (`nullptr` is false) and floats in `if`, `while`, `&&` and `||` conditions.
- Bugs this causes: `if (m[key])` in section 1 of `Notes_maps.md`, `if (s.find(...))` in section 5 of `Notes_strings.md`, and `isupper(c) == true` in section 7 of `Notes_strings.md`.

---

## 4. `v.begin()` and `v.end()`: where they point

- `v.begin()` points **to** the first element (index 0).
- `v.end()` points **one past** the last element, a slot that does not exist, so `*v.end()` is UB.
- Example: size 12 (valid indexes 0 to 11): `begin()` is index 0, `end()` is index 12 (an invalid slot).
- Iterator to the actual last element: `v.end() - 1` (random-access only) or `--v.end()`. For just the value, use `v.back()`.
- **Why "one past":** it lets `for (it = v.begin(); it != v.end(); ++it)` work cleanly, with no special case for the last element.
- Reverse iterators (`rbegin`, `rend`): section 1 of `Notes_vectors.md`.

---

## 5. Iterator operations

- `*it` dereferences: the value at that position.
- `it++` / `++it` moves to the next element, `it--` / `--it` to the previous one (bidirectional and better: `vector`, `list`, `set`, `map`).
- `it < jt` compares positions, on random-access iterators only. For `it + n` (also random-access only) and reverse iterators, see section 1 of `Notes_vectors.md`.
- `it == jt` and `it != jt` work for every iterator type. That is the standard `begin()` / `end()` loop comparison.
- `v.cbegin()` / `v.cend()` are const iterators: read-only.
- `cout << it` is invalid (iterators have no `<<`). Print `*it`.
- Prefer `auto it = v.begin();` over spelling out `Container::iterator`. It keeps code independent of the container.

---

## 6. Pointers vs references

**Pointer** (`int* p = &x;`): holds the address of `x`. `*p` dereferences to the value, and `&x` is the address-of operator.

**Reference** (`int& r = x;`): another name (an alias) for `x`. It must be bound when it is declared. Use `r` directly, with no `*`, and `&r == &x`.

| Property   | Pointer                                          | Reference                      |
|------------|--------------------------------------------------|--------------------------------|
| Reassign   | yes: `p = &y`                                    | no: bound once, at declaration |
| Null       | yes                                              | no                             |
| Arithmetic | yes: `p++`, `p + 1` move by `sizeof(type)` bytes | no: `r++` increments the value |
| Access     | needs explicit `*` (and `->`)                    | use `r` directly               |

- **Pointer pros:** nullable (works as "optional"), reassignable (iterators, linked structures), arithmetic (walking a buffer), required for `new` / `delete` and C APIs.
- **Pointer cons:** can be null or dangling (segfault, UB), easy to leak memory, verbose (`*` and `->` everywhere), no guarantee it points to a valid object.
- **Reference pros:** never null (safer), clean syntax, the idiomatic way to pass parameters (`void f(vector<int>& v)` avoids a copy), and a `const` reference avoids the copy while blocking changes.
- **Reference cons:** must be initialized when declared (no "declare now, bind later"), cannot be reseated (less flexible for re-pointing in linked lists and trees), no arithmetic.

A pointer and a reference to the same memory:
```cpp
int val = 100;
int* p = &val;          // p holds the address of val
int& ref = val;         // ref is just another name for val, with no separate address

// &val, p and &ref are all the same address: ref is an alias, not a new object
ref = 200;              // now val == 200 and *p == 200 too: same memory location
```

- **Key point:** a reference has no address of its own to inspect. `&ref` always equals the address of the thing it refers to.

---

## 7. The `&` operator: three meanings

1. **Address-of:** `int* p = &val;`. Applied to an existing variable or expression, it produces a pointer.
2. **Reference declarator:** `int& ref = val;`, `void f(vector<int>& v);`. It appears in a type, right after the type name, and only in declarations.
3. **Bitwise AND:** `int r = a & b;`. It sits between two integer values in an expression (section 1 of `Notes_bits.md`).

**How to tell them apart:** `&` right after a type name (`int&`, `T&`) is a reference type. `&` before an existing variable in an expression (`&val`) is address-of.

---

## 8. Binary search

**Invariant:** the answer lies in `[start, end]`, inclusive.

**Loop condition:** `while (start <= end)`, not `start < end`.
- `start <= end` means at least one element is left to check.
- With `start < end`, the loop stops when one element is left (`start == end`) and never checks it.
- Use `start < end` only for boundary searches like `lower_bound` ("first element `>=` target"). There the loop is meant to stop at `start == end`, because that position is the answer.

> TODO: Expand — the `start < end` version: where `end` starts, why it moves with `end = mid` (not `mid - 1`), and one `lower_bound` example.

**Middle:** `mid = start + (end - start) / 2`, not `(start + end) / 2`.
- `start + end` can overflow when both are near `INT_MAX`. `end - start` is at most the array size, so it cannot.
- Java's `binarySearch()` had this bug for years. It costs nothing, so always write it this way.

Standard template:
```cpp
int binarySearch(vector<int>& v, int target) {
    int start = 0, end = v.size() - 1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (v[mid] == target) return mid;
        else if (v[mid] < target) start = mid + 1;
        else end = mid - 1;
    }
    return -1;                                   // not found
}
```

---

## 9. Tree terms: ancestor, descendant, lowest common ancestor (LCA)

Example tree:
```
          A
        /   \
       B     C
      / \   / \
     D   E F   G
    /
   H
```

**Ancestor:** a node above you, on the way up to the root (parent, grandparent, ...). D's ancestors are B and A. Test: is it on the direct path from the node up to the root?

**Descendant:** a node below you, the opposite of an ancestor. A's descendants are all the other nodes in the tree.

**LCA:** the closest node that is an ancestor of both nodes: the point where their paths up to the root first meet.

LCA of each pair in the tree above:
```
D, E   -> B    (siblings: no need to go up to A)
D, F   -> A    (no shared parent, so they meet only at A)
D, H   -> D    (H is D's descendant, and a node counts as its own ancestor here)
```

- "Lowest" means closest, not furthest back. A is a common ancestor of nearly every pair, but it is the LCA only when no closer shared ancestor exists.
- Uses: the distance between two nodes, `dist(X, Y) = depth(X) + depth(Y) - 2 * depth(LCA)`, ancestor and descendant checks, and path-between-nodes problems.

Standard recursive LCA (binary tree, no parent pointers):
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (root == nullptr || root == p || root == q) return root;
    TreeNode* left  = lowestCommonAncestor(root->left, p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    if (left && right) return root;   // p, q split -> root is LCA
    return left ? left : right;        // same subtree, or one found
}
```

- **Intuition:** recurse into both sides. A node that gets a non-null answer from **both** calls has `p` and `q` split across it, so it is their LCA.

---

## 10. Dynamic programming: recursion -> memoization -> iterative

**When DP fits:** optimal substructure (the solution is built from subproblem solutions) plus overlapping subproblems (naive recursion computes the same subproblem many times).

Workflow, always in this order:
1. Write the brute-force recursion first. Do not jump to memoization.
2. Find the **state**: the smallest set of changing parameters that fully decides a subproblem's answer. The state's dimensions are the memo table's dimensions.
3. Check that overlap really exists. If it does not, memoization will not help.
4. Add the memo: check the cache before computing, store after computing.
5. Optionally convert to bottom-up (iterative) to avoid recursion overhead and stack overflow, then optimize space if only a fixed window of earlier states is needed.

Recursion -> memoization template (top-down):
```cpp
unordered_map<int,int> memo;   // or vector<int> memo(n+1, -1)

int solve(int n) {
    if (/* base case */) return base_value;
    if (memo.count(n)) return memo[n];   // check cache first
    memo[n] = /* recursive computation using solve(smaller n) */;
    return memo[n];
}
```

- The sentinel (`-1`) must be a value the real answer can never take. If answers can be negative, use a separate `visited[]` bool array instead.
- Time drops from exponential (for example O(2ⁿ) for naive Fibonacci) to O(n): each unique state is computed once, and repeats become O(1) cache hits.

Top-down vs bottom-up:

|             | Top-down                                          | Bottom-up                                        |
|-------------|---------------------------------------------------|--------------------------------------------------|
| Direction   | starts big, recurses down                         | starts at the base cases, iterates up            |
| Stack       | uses the call stack (overflow risk for large `n`) | no recursion, no stack risk                      |
| Deriving it | easier: add cache lines to the brute force        | needs the recurrence direction "flipped"         |
| Work done   | computes only the states actually needed          | computes every state up to `n`, even unused ones |

**Converting recursion to iteration is not "the same loop backward".** It means redefining what each table entry means, so the dependencies flow in the direction you iterate. Climbing Stairs:
- Top-down state `solve(idx)` = ways **from** `idx` **to** the end. It depends on future steps, so it is naturally recursive. Base case: `idx == n`.
- Bottom-up state `dp[idx]` = ways **from** the start **to** `idx`. It depends on past steps, so it is naturally iterative: loop forward from 0. Base cases: `idx == 0` / `1`.
- Same recurrence, mirrored direction. Both directions can be valid for symmetric recurrences. What matters: pick one meaning for `dp[idx]`, make sure its dependencies are already computed, and loop in whichever direction does that.

**Space optimization:** if `dp[i]` depends only on a fixed small window of earlier entries (such as `i-1` and `i-2`), replace the full array with a few variables, so O(n) space becomes O(1). This works only when the window is bounded. If `dp[i]` needs all earlier entries, keep the full table.

Common DP bugs:
- A wrong sentinel value (valid answers overlapping `-1`).
- Off-by-one in the memo size (`n+1` vs `n`).
- Memoizing on the wrong state: two different parameter combinations map to the same cell, giving silently wrong cached answers.
- A stale memo or map reused across separate top-level calls in a loop.

**The terms come in pairs** (DSA convention):
- Top-down == recursive + memoization
- Bottom-up == iterative + tabulation

---

## 11. Recursion direction: recursion itself is not top-down

Recursion is just a function calling itself. Whether it works top-down or bottom-up depends on **where the work happens relative to the recursive call** (before or after), not on recursion itself.

**Top-down recursion:** the work happens **before** recursing, and state is passed **down** to the children as parameters. Information flows root -> leaves, as in this sum along each root-to-leaf path:
```cpp
void topDown(TreeNode* root, int sum) {
    if (!root) return;
    sum += root->val;                  // work before recursing
    if (!root->left && !root->right) { use(sum); return; }
    topDown(root->left, sum);
    topDown(root->right, sum);          // pass state down
}
```

**Bottom-up recursion:** recurse **first**, and combine or use the results only **after** the calls return (you need the children's answers first). Information flows leaves -> root, as in this tree height:
```cpp
int bottomUp(TreeNode* root) {
    if (!root) return 0;
    int left = bottomUp(root->left);
    int right = bottomUp(root->right);  // wait for children
    return 1 + max(left, right);        // combine after
}
```

- Most tree aggregate problems (height, diameter, balanced check, path-sum aggregates) are naturally bottom-up.
- Problems that track a running value downward (the sum from the root to the current node, the largest ancestor seen so far) are naturally top-down.

**Naming clash between DP and trees:** DP's "top-down" (recursion + memo) has the same shape as tree "bottom-up recursion" above (recurse first, combine after). DP names it by which end you start reasoning from (top-down = start at the big question). Tree terminology names it by when the combine step happens (bottom-up = combine after). Same mechanics, different convention.

---

## 12. Tree recursion: the 3-question framework

**Q1. "What does one call to this function tell me?"** This is the **contract**: state it in one plain sentence before writing code. If you cannot state it, you are not ready to code.
- Example (height): "returns the height of the subtree rooted at this node."

**Q2. "What is the trivial case I already know without recursing?"** This is the **base case**: what breaks the pattern of having children to recurse into? Usually `nullptr`, sometimes specifically a leaf, depending on what the contract needs (see section 13).

**Q3. "If I had my children's answers, how do I build mine?"** This is the **combine** step, the recursive leap of faith: trust that the recursive calls already return correct answers per the contract. Do not unfold the whole call stack; just ask what to do with the left and right results.
- Example (height): "1 + the taller of my two children's heights" -> `return max(1 + left, 1 + right);`

**Q4. Only for global aggregates** (the largest diameter across the whole tree, not just at the root): "Is what I finally want the same as what I return to my parent, or a side computation to capture separately?"
- Same: just return it (Max Depth).
- Different: you need an **accumulator** that lives outside the normal return value (a member variable or a reference parameter), while the function still returns per the contract.
- Example (Diameter): the return value is the height (the parent needs it). Separately, `diameter = max(diameter, leftHeight + rightHeight)` is checked inside the function at **every** node, not just the root, because the longest path's peak can be any node.

Problem shapes:
- **Pure bottom-up value** (the parent's answer is a simple function of the children's, nothing else) -> just return normally. Max Depth, Path Sum (bool).
- **Global aggregate plus a local value** (need "best across the tree" and per-node info) -> accumulator + normal return per the contract. Diameter, Balanced check.
- **Information flows both ways** (down: a running value; up: the result) -> pass extra parameters down, return up. Path Sum with a running remaining total.

Common bugs:
- **Aggregate computed once:** using only the root's children instead of updating the accumulator inside the recursive function, so that every node is checked as a possible peak. Diameter must take `max(diameter, left + right)` at every node, not just once.
- **Discarded return value:** calling a pure function like `max(a, b)` and throwing the result away (`max(ans, x);` instead of `ans = max(ans, x);`). `max()` has no side effects: any function that returns a value must have that value assigned if you need it.

---

## 13. Leaf vs null: base cases in trees

**Null:** no node here at all (an empty subtree).

**Leaf:** a node exists here and has no children (`!root->left && !root->right`).

They get mixed up because a leaf's children **are** null, so recursing into a leaf's children hits the null base case. **The bug:** the null base case answers a question that only makes sense **at a leaf** (for example, "does this complete a valid root-to-leaf path?").

Before writing a base case, ask: "does this problem care about leaves specifically, or just empty subtrees?"
- **A separate leaf check is needed** when the problem says "leaf" and the answer depends on an accumulated condition (such as a running sum) that must hold exactly there.
- **No leaf check is needed** when the problem asks only about height, depth, count or structure: the null sentinel plus the combining formula usually handles leaves by itself (Max Depth, Diameter, Invert Tree, Count Good Nodes). `1 + max(left, right)` gives a leaf height 1 automatically, because its children hit the null sentinel.

Bug example, 112 Path Sum (a root-to-**leaf** path must sum to the target):
```cpp
// wrong: the null base case does leaf logic
bool solve(TreeNode* root, int targetSum) {
    if (!root) return targetSum == 0;   // fires on any null child, even of a
                                        // one-child node that isn't a leaf!
    ...
}
```

- It fails on `1 -> (right) 2` with `targetSum = 1`: `solve(null, 1 - 1 = 0)` wrongly returns true, but the only real path (1 -> 2, sum 3) does not match. The answer should be false.

The fixed version:
```cpp
// correct: an explicit leaf check, separate from the null check
bool hasPathSum(TreeNode* root, int targetSum) {
    if (!root) return false;
    if (!root->left && !root->right)
        return targetSum == root->val;   // the real leaf check
    bool left  = root->left  && hasPathSum(root->left,  targetSum - root->val);
    bool right = root->right && hasPathSum(root->right, targetSum - root->val);
    return left || right;
}
```

- Guard the recursive calls with `root->left &&` / `root->right &&` so that a missing child is never misread as a valid path ending.

**Technique:** write a base-case table before coding, as here for 112 Path Sum:
```
Empty subtree (root == nullptr)   -> false
Leaf (!left && !right)            -> targetSum == root->val
Internal (otherwise)              -> left || right
```

- If only the empty-subtree row is needed, you probably do not need a separate leaf case.

**Hand-trace these trees before trusting any tree recursion:** a single node (a leaf itself), a node with one real child and one null child (no leaf at the root), and a node with two leaf children. The one-real-one-null tree is the smallest case that exposes leaf-vs-null bugs, so always trace it first.

---

## 14. Majority element: the floor(n/2) detail

"Appears more than floor(n/2) times" means `count > floor(n/2)`, so `count >= floor(n/2) + 1`: strictly more than the **floored** half.

The smallest count that is a majority:
```
n = 6:  floor(6/2) = 3   -> need count >= 4
n = 7:  floor(7/2) = 3   -> need count >= 4   (3.5 is not reachable by an integer count anyway)
```

This guarantee (a true majority always wins any one-against-all cancelling) is **why Boyer-Moore Voting works**: no other element or mix of elements has enough occurrences to cancel the true majority out completely.

**Boyer-Moore Voting:** keep one candidate and a count. An element equal to the candidate adds 1, and any other element subtracts 1. When the count reaches 0, the current element becomes the new candidate with count 1. For 169 Majority Element:
```cpp
int majorityElement(vector<int>& nums) {
    int candidate = nums[0], count = 0;
    for (int x : nums) {
        if (x == candidate) count++; else count--;
        if (count == 0) { candidate = x; count = 1; }
    }
    return candidate;
}
```

- O(n) time, O(1) space.
- There is no second pass to check the count against n/2, because the problem (169) guarantees that a majority element exists. Without that guarantee the code still returns some value (`[1, 2, 3]` gives 3), so count the candidate in a second pass.

---

## 15. String reversal (STL)

- `reverse(s.begin(), s.end())` reverses **in place** and returns nothing (`void`). O(n) time, O(1) space. It works on a `string` the same as on a `vector`, since both support iterators.
- Partial reversal: `reverse(s.begin(), s.begin() + 5)` reverses only the first 5 characters.
- Reversed **copy**, leaving `s` untouched: `string reversed(s.rbegin(), s.rend());` builds a new string by walking `s` backward with reverse iterators (section 1 of `Notes_vectors.md`).

---

## 16. Char literal vs int: `'0'` is not `0`

`'0'` is a **char literal** with numeric (ASCII) value 48, not 0. `int carry = '0';` quietly sets `carry` to 48, which is easy to miss because it "looks like" zero-initialization.
- Always convert a digit character to its number with `c - '0'` (gives 0 to 9). Never treat a bare char literal as the integer 0 in arithmetic.
- ASCII values and `char` arithmetic: section 1 of `Notes_strings.md`.

Bug example, 67 Add Binary:
```cpp
int carry = '0';   // wrong: starts at 48
int carry = 0;     // correct
```

- The bogus 48 is added again on every iteration through `sum = carry + ...`, producing garbage multi-digit output instead of the correct binary sum.

Reusable digit-string addition (Add Binary with `base = 2`, Add Strings with `base = 10`, and so on):
```cpp
while (i >= 0 || j >= 0 || carry) {
    int sum = carry;
    if (i >= 0) sum += a[i--] - '0';
    if (j >= 0) sum += b[j--] - '0';
    ans.push_back('0' + (sum % base));
    carry = sum / base;
}
```

- Folding `|| carry` into the loop condition handles a final carry (`"1" + "1" = "10"`) automatically, with no separate check after the loop.
- It also removes the need to swap `a` and `b` by length: the `i >= 0` / `j >= 0` guards handle unequal lengths.
- The digits come out last first, so reverse `ans` once at the end ("Remember" in section 8 of `Notes_strings.md`).

---

## 17. Loop boundary off-by-one: `> 0` vs `>= 0`

`while (i > 0 || j > 0)` stops **before** processing index 0. But index 0 is still valid, meaningful data (the last item to process), not a sentinel to exclude.
- Walking indexes down to and including 0: `while (i >= 0 || j >= 0)` (plus `|| carry` if needed, see section 16).
- The real "nothing left" state is when the index goes **negative** (`< 0`), not when it reaches 0. Index 0 is the last valid index of anything.
- Always double-check `>` vs `>=` when looping down to array or string index 0.

---

## 18. Slow and fast pointers: start positions and the math

Both `slow` and `fast` start at the **same** position, `head`, not offset from each other:
```cpp
ListNode* slow = head; ListNode* fast = head;
```

**Why the same start matters:** the technique relies on a fixed speed ratio (fast moves 2 nodes per step, slow moves 1). Starting together at that ratio guarantees that when fast finishes, slow has covered exactly half the distance. Offsetting the start (for example `fast = head->next`) keeps the 2:1 step ratio but moves slow's final position off the true middle.

**The math:** the list has `n` nodes. After `k` steps, fast has moved `2k` nodes and slow `k`. The loop ends when fast reaches the end: `2k ≈ n`, so `k ≈ n/2`. Slow moved `n/2` nodes from the same start, so it ends at the middle.
- General formula: if fast moves `p` steps for every `q` steps of slow, slow ends at position `n * (q/p)` when fast finishes. For 2:1 (`p = 2`, `q = 1`) that is `n/2`. A 3:1 ratio ends slow at the 1/3 point, which some "k-th fraction point" variations use.

Standard template, 876 Middle of the Linked List:
```cpp
ListNode* middleNode(ListNode* head) {
    ListNode* slow = head, *fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```

Odd length, `n = 5` (indexes 0 to 4, true middle = index 2):
```
step 1   fast 0 -> 2, slow 0 -> 1
step 2   fast 2 -> 4, slow 1 -> 2
check    fast = 4, fast->next = null -> stop
slow = index 2: the exact middle
```

Even length, `n = 6` (indexes 0 to 5; LeetCode wants the **second** middle, index 3):
```
step 1   fast 0 -> 2, slow 0 -> 1   (continue, fast->next = 3 is valid)
step 2   fast 2 -> 4, slow 1 -> 2   (continue, fast->next = 5 is valid)
step 3   fast 4 -> 6 (null), slow 2 -> 3
check    fast = null -> stop
slow = index 3: correct
```

**Why the loop needs both conditions** (`fast != nullptr && fast->next != nullptr`):
- `fast->next != nullptr` alone crashes on even lengths: `fast` itself can be null, and then `fast->next` is UB.
- `fast != nullptr` alone crashes on odd lengths: `fast->next` can be null while `fast` is not, and then `fast->next->next` dereferences null.
- Both together handle both parities safely.

---

## 19. Bit manipulation: extra uses and explanations

The operators (`&`, `|`, `^`, `~`, `<<`, `>>`) with their properties, precedence, get / set / clear / toggle bit `i`, the tricks, Single Number, Missing Number, swap without a temporary, and bitmask subsets are in `Notes_bits.md`. This section keeps what that file does not have.

Truth table:
```
(a, b)    AND   OR   XOR
(0, 0)     0     0    0
(0, 1)     0     1    1
(1, 0)     0     1    1
(1, 1)     1     1    0
```

All three operators on two numbers, `a = 12` and `b = 10`:
```
a     = 12 = 1100
b     = 10 = 1010
a & b = 1000 = 8     only bit position 3 has a 1 in both
a | b = 1110 = 14    a bit is 1 if either has a 1: positions 1, 2, 3
a ^ b = 0110 = 6     a bit is 1 only where they differ: positions 1, 2
                     (position 3 matches (1, 1), so it is 0)
```

What each one is for:
- `a & b`: checking which bits are common to (shared by) two flag sets.
- `a | b`: merging two flag sets (union), for example `flags = FLAG_A | FLAG_B`.
- `a ^ b`: finding which bits differ between two states (spotting a toggled or changed bit), or isolating the value left over after matching pairs cancel.

**`~` flips every bit in the full width, including the sign bit.** `~5` on a 32-bit `int` is `-6` (two's complement), not a small positive number. Do not expect it to invert only the bits you care about.

Why the common patterns work:
- **Power of 2,** `n > 0 && (n & (n - 1)) == 0`: a power of 2 has a single 1-bit. `n - 1` turns that bit off and every bit after it on, so the AND is 0 only if there was exactly one set bit.
- **Count set bits,** `while (n) { n &= (n - 1); count++; }`: each pass clears the lowest set bit, so it runs O(number of set bits) times. `__builtin_popcount(n)` (GCC / Clang) is close to O(1), backed by hardware.
- **Missing Number (268):** XOR-ing the full expected range against the actual values cancels everything except the missing number.

**DSA uses, even though bits feel like a separate topic:** Single Number I / II / III, Counting Bits, Power of Two / Four, subsets by bitmask enumeration (bit `i` = include element `i` or not, for masks 0 to `2ⁿ - 1`), and DP with a bitmask state (for example, traveling-salesman-style problems).

Common bugs:
- Right-shifting a negative number is implementation-defined before C++20 (the exact rounding), but an arithmetic (sign-keeping) shift is universal in practice.
- `1 << 31` does not give 2³¹ in a 32-bit `int`. Since C++20 it gives `INT_MIN`, a negative number. Before C++20 it was undefined or implementation-defined, depending on the standard version. Use `1u << 31` or `1LL << 31` if you need 2³¹ (see "Overflow" in section 7 of `Notes_bits.md`).

---

## 20. `lower_bound` and `upper_bound`

- `lower_bound(target)`: the first element `>= target`.
- `upper_bound(target)`: the first element `> target` (strictly greater, so an exact match is excluded).

Sorted `{10, 20, 30, 40}`:
```
lower_bound(20)   -> 20     (exact match included)
upper_bound(20)   -> 30     (exact match skipped, next bigger)
lower_bound(25)   -> 30
upper_bound(25)   -> 30     (identical when there is no exact match: both return the next bigger element)
```

Two forms:
- Member functions on `set`, `map` and `multiset`: O(log n), using the tree structure directly.
- Free functions from `<algorithm>` on a sorted `vector` or array: `lower_bound(v.begin(), v.end(), x)`, an O(log n) binary search. On an unsorted range they give a silently **wrong** answer, not a crash, so always make sure the range is sorted first.

> TODO: Expand — what the free function `lower_bound(s.begin(), s.end(), x)` costs on a `set`, compared with the member `s.lower_bound(x)`.

- Always check `it != s.end()` (or `!= v.end()`) before dereferencing. If the target is bigger than everything, both return `end()`.
- **To remember:** `lower_bound` = the lower (left) boundary of where the target fits, so exact matches are included. `upper_bound` = the upper (right) boundary, the position just past all elements equal to the target, so exact matches are skipped.

---

## 21. Sort comparator: strict `<`, never `<=`

`std::sort` (and `priority_queue`, `set`, `map`, ...) requires the comparator to be a **strict weak ordering**: `comp(x, x)` must always return false for any element `x`. This is a hard mathematical requirement that the algorithm's internals depend on, not a style choice.

Using `<=` breaks it, because `comp(x, x)` becomes `x.second <= x.second`, which is always true:
```cpp
// wrong: violates strict weak ordering
sort(dist.begin(), dist.end(), [](pair<int,int> &a, pair<int,int> &b){
    return a.second <= b.second;
});

// correct
return a.second < b.second;
```

**Why it "works" on small inputs but crashes or misbehaves on large ones:** introsort (the quicksort / heapsort / insertion-sort hybrid that `std::sort` uses) relies on the comparator's guarantees to decide partitioning and recursion. With a broken comparator, `a < b` and `b < a` can both look true for equal elements. On large inputs with many duplicate keys this leads to out-of-bounds access inside the array: a segfault or heap corruption (libstdc++), not just a wrong order. Small inputs may not happen to trigger the bad access pattern.

`<=` cannot be rescued by lambda captures, `const&` parameters or any rewrite. The problem is the math of `<=` itself (`x <= x` is always true), not syntax, so there is no valid way to use `<=` in a comparator.
- **Goal: ties in any order are fine.** Plain `<` already allows this; `sort` guarantees no particular order among equal elements.
- **Goal: keep the original relative order among ties.** Use `stable_sort` instead of `sort`, still with strict `<`.
- **Goal: an explicit tie-breaker** (sort by distance, then by original index). Use a compound comparator with strict `<` at every level:
```cpp
sort(dist.begin(), dist.end(), [](const pair<int,int> &a, const pair<int,int> &b){
    if (a.second != b.second) return a.second < b.second;
    return a.first < b.first;      // tie-break by original index
});
```

**General rule:** every custom comparator passed to `sort`, `priority_queue`, `set` or `map` must use strict `<` (or `>` for descending), never `<=` or `>=`.

---

## 22. Priority queue / heap: concepts and extras

Header `<queue>`: `priority_queue` lives there, not in a header named after itself. The working notes are in `Notes_heap.md`: what a heap is, the vector layout and index formulas, push and pop step by step, complexity, syntax for every type, custom comparators, top K, heap vs `set`, and traps. This section keeps what that file does not have.

Heap vs priority queue, concept vs implementation:
- **Heap:** a technique, not a class name. A complete binary tree (every level full except possibly the last, which fills left to right) that satisfies the heap property: every parent `>=` both children (max-heap) or `<=` both children (min-heap).
- **Priority queue:** an abstract data type, a behavior contract: "give me the highest-priority element fast, let me insert new elements, let me remove the highest-priority one". It could be built several ways:
```
unsorted array:   find-max O(n),   insert O(1)
sorted array:     find-max O(1),   insert O(n)
binary heap:      find-max O(1),   insert and remove O(log n)    <- the best balance
```

- That balance is why `priority_queue` uses a heap internally.
- The relationship is the same as `vector` (the concrete class) implementing a dynamic array (the concept): `priority_queue` is the concrete STL class, and the heap is the technique it uses inside.

**Why a flat vector, not a pointer-based tree:** contiguous memory is cache-friendly, there is no per-node pointer overhead, and appending or shrinking at the end is O(1) amortized, which `push` and `pop` rely on.

**The `Container` argument** (the 2nd template argument) is the underlying storage. It does not choose max-heap vs min-heap. It must support random access plus `push_back`, `pop_back` and `front`: `vector` and `deque` qualify, `list` does not (no random access). In practice, always use `vector<T>` or leave the default.
- You can put any type in a heap, including `pair<int,int>` or your own `struct`, as long as it has `<` defined or you supply a comparator.

**The compile error for a two-argument min-heap:** in `priority_queue<int, greater<int>>`, the compiler reads `greater<int>` as the container (the 2nd slot). It fails because `greater<int>` is a comparator with no `value_type`, and the error is `no type named 'value_type' in 'std::greater<int>'`. Fix: always write all three arguments for a min-heap.

Methods compared with `queue` and `stack`:

| Method    | `queue` | `priority_queue` | `stack` | Notes                                                     |
|-----------|---------|------------------|---------|-----------------------------------------------------------|
| `push(x)` | yes     | yes              | yes     | insert: O(1) for `queue` / `stack`, O(log n) for the heap |
| `pop()`   | yes     | yes              | yes     | returns nothing (`void`) in all three                     |
| `front()` | yes     | no               | no      | `queue` only                                              |
| `back()`  | yes     | no               | no      | `queue` only                                              |
| `top()`   | no      | yes              | yes     | `priority_queue` and `stack` use `top()`, not `front()`   |
| `empty()` | yes     | yes              | yes     |                                                           |
| `size()`  | yes     | yes              | yes     |                                                           |

- To inspect every element of a `priority_queue`, you must pop everything (destroying the heap) or use a different container.

Which structure for "repeatedly get the max / min":

| Structure                  | Find max / min  | Insert                          | Remove max / min                                 |
|----------------------------|-----------------|---------------------------------|--------------------------------------------------|
| Unsorted vector            | O(n) scan       | O(1) `push_back`                | O(n) (find, then erase)                          |
| Sorted vector, kept sorted | O(1) (the ends) | O(n) (shift to insert in place) | O(n) shift, or O(1) if only removing from an end |
| `priority_queue` (heap)    | O(1) `top()`    | O(log n)                        | O(log n)                                         |

- **Unsorted vector** is best for: rarely, one-off max lookups only.
- **Sorted vector** is best for: static data, few inserts, many range queries (it also gives `lower_bound` / `upper_bound`, which a heap does not).
- **Heap** is best for: repeated inserts and removals of the max / min mixed together, the classic top K, streaming and "always need the current best" workloads.
- If you need the max only **once**, with no more inserts, even an unsorted vector and one O(n) scan (`max_element`) beats building a heap for no reason.

Common bug not covered in `Notes_heap.md`:
- Before C++20, `decltype(cmp)` needs the lambda declared first as a variable, and `pq(cmp)` passes it in. You cannot write the lambda directly in the template argument list. Since C++20 you can: C++20 allows a lambda inside `decltype` and makes a capture-less lambda default-constructible, so this min-heap needs no `cmp` argument:
```cpp
priority_queue<int, vector<int>, decltype([](int a, int b) { return a > b; })> pq;
```

---

## 23. Inorder traversal: iterative version and BST validation

**Inorder traversal:** visit the left subtree, then the current node, then the right subtree, applied recursively at every node. For a binary search tree (BST), inorder visits the nodes in **sorted ascending order**. That property is what makes it useful for BST validation (below).

Recursive version, for contrast:
```cpp
void inorder(TreeNode* root, vector<int>& ans) {
    if (!root) return;
    inorder(root->left, ans);
    ans.push_back(root->val);
    inorder(root->right, ans);
}
```

Iterative version, using an explicit stack instead of the call stack:
```cpp
void iterative(TreeNode* curr, vector<int>& ans) {
    stack<TreeNode*> st;

    while (curr || !st.empty()) {
        // curr indicates the node that needs to be processed.
        // we validate it and then add it to the stack
        while (curr) {
            st.push(curr);
            curr = curr->left;
        }
        TreeNode* node = st.top(); st.pop();
        ans.push_back(node->val);
        curr = node->right;
    }
}
```

How to read this logic:
- The inner `while` walks all the way down the **left spine** from `curr`, pushing every node on the way onto the stack. Processing is deferred: "I'll come back to you after I've gone as far left as possible."
- Once `curr` becomes null (no more left children), the leftmost unprocessed node is on top of the stack. Pop it: that is the next node to **visit** (record its value).
- After visiting a node, move to its **right** subtree (`curr = node->right`) and let the outer loop repeat. If that right subtree has its own left spine, the inner loop walks down it again before the next visit.
- The outer condition `curr || !st.empty()` covers both "there is still a subtree to walk down" (`curr` non-null) and "there is still a deferred node waiting to be visited" (stack non-empty). The traversal is not done until both are exhausted.
- This mirrors the recursive version exactly: the stack plays by hand the role the call stack plays automatically in the recursive version (remembering "go back to this parent after finishing its left subtree").

Time O(n): every node is pushed and popped exactly once. Space O(h) for the explicit stack (`h` = tree height), the same as the recursion depth of the recursive version.

**Use: 98 Validate Binary Search Tree.** Inorder traversal of a valid BST produces values in **strictly ascending** order. So validating a BST reduces to: perform an inorder traversal, and check that each visited value is strictly greater than the previous one. If any value is `<=` the previous one, the tree is not a valid BST.

The iterative skeleton with that check added:
```cpp
bool isValidBST(TreeNode* root) {
    stack<TreeNode*> st;
    TreeNode* curr = root;
    long long prev = LLONG_MIN;   // sentinel smaller than any valid node value

    while (curr || !st.empty()) {
        while (curr) {
            st.push(curr);
            curr = curr->left;
        }
        TreeNode* node = st.top(); st.pop();
        if (node->val <= prev) return false;   // must be strictly increasing
        prev = node->val;
        curr = node->right;
    }
    return true;
}
```

- **Why this works:** it is the exact same iterative inorder skeleton, with one check added at the visit step. Instead of just recording the value, compare it with the previously visited value. Inorder visits nodes in the exact order a valid BST requires them to be sorted, so a single linear pass is enough: no need to build a separate array and sort or check it afterward.
- A 64-bit sentinel (not `INT_MIN`) stops a valid node whose value is `INT_MIN` from failing the check, since the problem's node values can reach `INT_MIN` / `INT_MAX`.
- Use `long long`, not `long`. `long` is only 32 bits on Windows (MSVC), where `LONG_MIN` equals `INT_MIN`, so a node with value `INT_MIN` would wrongly fail. `long long` is 64 bits on every platform.

---

## 24. Graph / grid BFS: mark visited at push time, not pop time

**BFS vs DFS, the core difference: the order you visit nodes in.** BFS uses a **queue** to visit the **closest** nodes first, exploring level by level, outward in rings from the start. DFS skips past nearby nodes and follows a single path all the way until it hits a dead end, before backtracking to try others.

**The classic bug ("delayed marking"):** marking a cell or node visited when it is **popped** from the queue, instead of when it is **pushed**.

The bug in code:
```cpp
// wrong: marks on pop
while (!q.empty()) {
    auto [i, j] = q.front(); q.pop();
    grid[i][j] = VISITED;          // <-- too late
    for (each neighbor (nx, ny))
        if (grid[nx][ny] == UNVISITED)
            q.push({nx, ny});
}
```

- **Why it is wrong:** a cell can have several unvisited neighbors in the queue at the same time, all pointing back to it. The cell is not marked visited until it is popped, so every one of those neighbors sees it as still unvisited and pushes it **again** before it is ever processed. The same cell ends up in the queue many times.
- **Example:** a 2x2 block of connected land. `(1,1)` has two neighbors, `(0,1)` and `(1,0)`. If you mark on pop, both can push `(1,1)` before its first copy is popped.
- **Impact:** the queue grows well past O(m·n) with duplicate entries for the same cells, turning what should be an O(m·n) BFS into something far slower. This is a classic cause of TLE on dense or large grids in flood-fill problems (for example 200 Number of Islands). It can also cause double counting in problems that tally something per visited cell or node.

**Fix: mark visited at push time**, including the very first (starting) node:
```cpp
q.push({start_i, start_j});
grid[start_i][start_j] = VISITED;      // mark immediately

while (!q.empty()) {
    auto [i, j] = q.front(); q.pop();
    for (each neighbor (nx, ny))
        if (grid[nx][ny] == UNVISITED) {
            grid[nx][ny] = VISITED;     // mark before pushing
            q.push({nx, ny});
        }
}
```

- Now each cell is pushed exactly once: true O(m·n) time, and O(m·n) worst-case space (the largest queue, for example a solid block of land).

**Rule of thumb:** in BFS, set "visited" at the moment a node or cell is **added** to the queue, never later when it is removed. This applies to any BFS over a graph or an implicit grid graph, not just island counting: the same bug and fix show up in shortest-path BFS, multi-source BFS, and so on.

---

## 25. Graph representations: adjacency list vs adjacency matrix

**`V`:** the number of vertices (nodes): `n` in most problem statements.

**`E`:** the number of edges (connections): `edges.size()` in most problems.

### Adjacency matrix: a 2D array

Building one:
```cpp
vector<vector<int>> graph(n, vector<int>(n, 0));
graph[u][v] = 1;  // edge from u to v
```

- **Structure:** an `n × n` matrix where `graph[i][j] = 1` means the edge `i -> j` exists and `0` means no edge. For undirected graphs it is symmetric (`graph[i][j] == graph[j][i]`).
- **Space O(V²):** it **always** allocates `n²` cells, whatever the actual edge count. Example: 10,000 nodes with 100 edges still allocate 100,000,000 cells, using only 200 of them.
- **Check if an edge exists:** O(1), direct array access `graph[u][v]`.
- **Get all neighbors of vertex `i`:** O(V), because you must scan the entire row (code below).
- **Add an edge:** O(1).
- **BFS / DFS neighbor iteration:** O(V) per vertex, so O(V²) in total, even if only E edges exist.

Scanning a row for neighbors:
```cpp
for (int j = 0; j < n; j++)
    if (graph[i][j] == 1) { /* j is a neighbor */ }
```

Use cases (rare):
- Dense graphs, where `E ≈ V²` (nearly every pair connected).
- When you need O(1) edge-existence checks very frequently.
- Algorithms such as Floyd-Warshall (all-pairs shortest paths) that iterate over all `V²` pairs anyway.

### Adjacency list: a vector of vectors

Building one:
```cpp
vector<vector<int>> graph(n);  // n empty vectors
graph[u].push_back(v);         // add edge u->v
graph[v].push_back(u);         // undirected: add reverse too
```

- **Structure:** an array of `V` dynamic lists. `graph[i]` holds only the **actual** neighbors of vertex `i`.
- **Space O(V + E):** the outer structure is `V` vectors (one per vertex, possibly empty). The inner elements across all vectors total `2E` for undirected graphs (each edge is stored twice) or `E` for directed ones.

Breakdown for `V = 5`, `E = 6`:
```
graph[0] = [1, 2]       // 2 elements
graph[1] = [0, 3]       // 2 elements
graph[2] = [0, 3, 4]    // 3 elements
graph[3] = [1, 2, 4]    // 3 elements
graph[4] = [2, 3]       // 2 elements
Total: 5 (outer) + 12 (inner) = 17 units
```

- **Why "V + E", not "V × E":** you **add** the space for the structure (`V` vectors) and the space for the data (the sum of all inner sizes, `2E`). In a matrix, every one of the `V` rows has `V` columns, so you multiply. In a list, each of the `V` vectors has a **different** size (its degree), and the sum of those sizes equals the total edge count.
- **Check if an edge exists:** O(degree of `u`), a linear search in `graph[u]` for `v` (O(V) in the worst case, for a hub node). Use `unordered_set<int>` instead of `vector<int>` for O(1) average lookup if needed.
- **Get all neighbors of vertex `i`:** O(degree of `i`), iterating only `graph[i]`: `for (int neighbor : graph[i]) { /* process */ }`.
- **Add an edge:** O(1) amortized (`vector` `push_back`).
- **BFS / DFS neighbor iteration:** O(degree) per vertex, so O(E) in total across the entire traversal (each edge checked at most twice for undirected graphs).

Use cases (99% of graph problems):
- Sparse graphs (`E << V²`): real-world graphs are nearly always sparse.
- BFS, DFS and shortest-path algorithms (Dijkstra, Bellman-Ford).
- Most LeetCode and interview graph problems.

### Comparison

| Operation         | Adjacency matrix | Adjacency list   |
|-------------------|------------------|------------------|
| Space             | O(V²)            | O(V + E)         |
| Check edge exists | O(1)             | O(degree) / O(V) |
| Get all neighbors | O(V)             | O(degree)        |
| Add edge          | O(1)             | O(1)             |
| BFS / DFS total   | O(V²)            | O(V + E)         |

Real-world impact (LeetCode constraint example: `V ≤ 2×10⁵`, `E ≤ 2×10⁵`):
```
Matrix:   2×10⁵ × 2×10⁵ = 4×10¹⁰ ints ≈ 160 GB   (impossible: out of memory)
List:     2×10⁵ + 4×10⁵ = 6×10⁵ ints ≈ 2.4 MB    (fits easily)
```

Standard template, 1971 Find if Path Exists in Graph:
```cpp
bool validPath(int n, vector<vector<int>>& edges, int source, int destination) {
    if (source == destination) return true;

    // Build adjacency list: O(V + E) space
    vector<vector<int>> graph(n);
    for (auto& edge : edges) {
        graph[edge[0]].push_back(edge[1]);
        graph[edge[1]].push_back(edge[0]);
    }

    // BFS
    queue<int> q;
    vector<bool> visited(n, false);

    q.push(source);
    visited[source] = true;

    while (!q.empty()) {
        int curr = q.front();
        q.pop();

        if (curr == destination) return true;

        // Only iterate actual neighbors: O(degree) per vertex
        for (int neighbor : graph[curr]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;  // mark at push time (see section 24)
                q.push(neighbor);
            }
        }
    }

    return false;
}
```

Key optimizations vs a naive matrix approach:
1. **Visited tracking:** a separate bool array instead of modifying the graph in place (`graph[i][j] = -1` pollutes the structure).
2. **Neighbor iteration:** a for-each loop over `graph[curr]` instead of scanning all `n` vertices with `for (int j = 0; j < n; j++)`.
3. **Early termination:** return as soon as the destination is found, instead of waiting to exhaust the queue.

Common bugs:
- Forgetting to add **both** directions for undirected graphs (`graph[u].push_back(v)` **and** `graph[v].push_back(u)`).
- Using `graph.size()` when you meant `graph[i].size()` (outer size vs inner size).
- Trying to access `graph[u][v]` like a matrix when `graph[u]` is a list: a compile error or a logic bug (`v` is an index into the neighbor list, not a vertex-ID check).
- Not initializing the visited array, or reusing a stale one across several BFS / DFS calls in a loop.
