# Bit Notes (DSA interviews)

**Abbreviations**
- **BODMAS:** Brackets, Orders, Division, Multiplication, Addition, Subtraction
- **XOR:** exclusive OR

For the letter case tricks (`| 32`, `& ~32`, `^ 32`), see "Why 32 works" in `Notes_strings.md`.

## 1. Operators

To print bits while debugging: `cout << bitset<8>(5);` prints `00000101` (header `<bitset>`).

### `&` AND

Compares two numbers bit by bit. A result bit is 1 only if both bits are 1.
- `x & 0 = 0` and `x & x = x`.
- `x & 1` is the lowest bit: 1 if `x` is odd, 0 if even.
- `x & mask` keeps only the bits that are 1 in `mask` and clears the rest.

Examples:
```
5 & 3      -> 1     (0101 & 0011 = 0001)
12 & 10    -> 8     (1100 & 1010 = 1000)
```

### `|` OR

Compares two numbers bit by bit. A result bit is 1 if either bit is 1.
- `x | 0 = x` and `x | x = x`.
- `x | mask` sets the bits that are 1 in `mask` and leaves the rest unchanged.

Examples:
```
5 | 3          -> 7     (0101 | 0011 = 0111)
12 | 10        -> 14    (1100 | 1010 = 1110)
5 | (1 << 1)   -> 7     (set bit 1: 0101 -> 0111)
8 | 0          -> 8
```

### `^` XOR (exclusive OR)

Compares two numbers bit by bit. Same bits give 0, different bits give 1.
- `x ^ 0 = x` and `x ^ x = 0`.
- The order does not matter: `a ^ b ^ a = b`, so pairs cancel out.
- Applying the same value twice undoes it: `(x ^ k) ^ k = x`.
- `x ^ mask` flips the bits that are 1 in `mask`.

Examples:
```
5 ^ 3          -> 6     (0101 ^ 0011 = 0110)
12 ^ 10        -> 6     (1100 ^ 1010 = 0110)
7 ^ 7          -> 0
7 ^ 0          -> 7
(5 ^ 3) ^ 3    -> 5
```

### `~` NOT

Takes one number and flips every bit: 0 becomes 1 and 1 becomes 0.
- For an `int`, `~x = -x - 1`.
- `~~x = x`.
- `x & ~mask` clears the bits that are 1 in `mask`.

Examples:
```
~5         -> -6    (0000 0101 -> 1111 1010)
~0         -> -1
~(-1)      -> 0
5 & ~1     -> 4     (clear bit 0: 0101 -> 0100)
```

### `<<` Left shift

`n << k` moves every bit of `n` k places to the left. The empty places on the right fill with 0, and bits pushed off the left are lost.
- `n << k = n * 2^k`, as long as the result fits.
- `1 << k = 2^k`, a number with only bit `k` set.
- The result has the type of `n`: `1 << k` is an `int`, and `1LL << k` is a `long long`.

Examples:
```
5 << 1       -> 10              (0000 0101 -> 0000 1010)
5 << 2       -> 20              (0000 0101 -> 0001 0100)
1 << 4       -> 16              (0000 0001 -> 0001 0000)
3 << 3       -> 24
1LL << 40    -> 1099511627776
```

### `>>` Right shift

`n >> k` moves every bit of `n` k places to the right. Bits pushed off the right are lost, and for a non-negative `n` the left fills with 0.
- `n >> k = n / 2^k`, rounded down, for a non-negative `n`.
- `(n >> k) & 1` reads bit `k`.
- For a negative `n` the sign bit is copied in on the left, so `>>` rounds down while `/` rounds toward 0. `-7 >> 1` and `-7 / 2` give different answers.

Examples:
```
20 >> 2         -> 5       (0001 0100 -> 0000 0101)
5 >> 1          -> 2       (0000 0101 -> 0000 0010)
1 >> 1          -> 0
(13 >> 2) & 1   -> 1       (13 = 1101, bit 2 is 1)
-7 >> 1         -> -4      (but -7 / 2 -> -3)
```

---

## 2. Operator precedence (BODMAS for C++)

**C++ precedence is BODMAS (Brackets, Orders, Division, Multiplication, Addition, Subtraction) from school, with extra levels added below it.** The full order, highest first:

| Level | Operators              | Name                   |
|-------|------------------------|------------------------|
| 1     | `()` `[]`              | brackets (B)           |
| 2     | `-a` `!` `~` `++` `--` | unary (O)              |
| 3     | `*` `/` `%`            | multiply, divide (D M) |
| 4     | `+` `-`                | add, subtract (A S)    |
| 5     | `<<` `>>`              | shifts                 |
| 6     | `<` `<=` `>` `>=`      | comparison             |
| 7     | `==` `!=`              | equality               |
| 8     | `&`                    | bitwise AND            |
| 9     | `^`                    | bitwise XOR            |
| 10    | `\|`                   | bitwise OR             |
| 11    | `&&`                   | logical AND            |
| 12    | `\|\|`                 | logical OR             |
| 13    | `?:` `=` `+=` `&=`     | ternary, assignment    |

- B, D M and A S work as in school: brackets first, then D and M at one level, then A and S at one level, each left to right.
- O (orders, powers) has no operator in C++. The unary operators sit at its level.
- School maths stops at A S. Levels 5 to 13 are the extra C++ levels.
- Operators on the same level go left to right, except unary operators and assignment, which go right to left. `a = b = 5` sets both to 5.

**To remember:** BODMAS, then shifts, then compare (`<`, then `==`), then bitwise (`&` `^` `|`), then logical (`&&` `||`), then assignment.

BODMAS rules that work the same in C++:
```
2 + 3 * 4       -> 14      (M before A)
(2 + 3) * 4     -> 20      (B first)
8 / 2 * 4       -> 16      (D and M are the same level, so left to right)
10 - 4 - 3      -> 3       (A and S are the same level, so left to right)
-2 * 3          -> -6      (unary minus first)
```

Levels below BODMAS (these cause the bugs in section 7):
```
n & n - 1       -> n & (n - 1)      (arithmetic before &, so 12 gives 8)
1 << k - 1      -> 1 << (k - 1)     (k = 3 gives 4, not 7)
(1 << k) - 1    -> 7                (k = 3, a mask of k ones)
3 + 5 >> 1      -> 4                ((3 + 5) >> 1)
4 | 6 & 1       -> 4                (4 | (6 & 1), & before |)
2 ^ 3           -> 1                (^ is XOR, not power)
```

---

## 3. Single-bit operations

Bit `k` counts from the right, starting at 0. The four operations on `n = 5`:
```
n = 5 (0101)

(n >> 2) & 1       -> 1   (is bit 2 set?)
n | (1 << 1)       -> 7   (set bit 1)
n & ~(1 << 2)      -> 1   (clear bit 2)
n ^ (1 << 0)       -> 4   (toggle bit 0)
```

---

## 4. Tricks for DSA

The tricks, with the rule behind each in brackets:
```
7 & 1                        -> 1       (odd)
8 & 1                        -> 0       (even)
12 & 11                      -> 8       (n & (n - 1) removes the lowest set bit, n = 12)
12 & -12                     -> 4       (n & -n isolates the lowest set bit)
16, 18 power of two?         -> true, false   (n > 0 && (n & (n - 1)) == 0)
__builtin_popcount(13)       -> 3       (13 = 1101 has three set bits)
```

Count set bits by hand (each pass removes the lowest set bit):
```cpp
int count = 0;
while (n) { n &= n - 1; count++; }     // n = 13 gives 3
```

- `__builtin_popcount` works with GCC and clang. Use `__builtin_popcountll` for `long long`. C++20 also has `std::popcount` in `<bit>`, but it only takes unsigned types: `popcount(13u)` works, and `popcount(13)` with an `int` does not compile.

---

## 5. XOR problems

`a ^ a = 0` cancels pairs, so XOR-ing everything leaves the value that has no pair.

### Single number

Every value appears twice except one:
```cpp
int x = 0;
for (int v : nums) x ^= v;
```
```
[4, 1, 2, 1, 2]   -> 4
```

### Missing number

Values 0..n with one missing:
```cpp
int x = n;
for (int i = 0; i < n; i++) x ^= i ^ nums[i];
```
```
n = 3, [3, 0, 1]  -> 2
```

### Swap without a temporary

An interview trick (use `std::swap` in real code):
```cpp
a ^= b;  b ^= a;  a ^= b;
```
```
a = 3, b = 5      -> a = 5, b = 3
```

- `a` and `b` must be two different variables: see "XOR swap on one variable" in section 7.

---

## 6. Bitmask subsets

Bit `i` of `mask` says whether element `i` is in the subset. Looping `mask` from 0 to `(1 << n) - 1` visits all 2ⁿ subsets.
```cpp
for (int mask = 0; mask < (1 << n); mask++)
    for (int i = 0; i < n; i++)
        if (mask & (1 << i)) { /* element i is in this subset */ }
```
```
elements {a, b, c}, n = 3

mask 0 -> {}         mask 4 -> {c}
mask 1 -> {a}        mask 5 -> {a, c}
mask 2 -> {b}        mask 6 -> {b, c}
mask 3 -> {a, b}     mask 7 -> {a, b, c}
```

- It is practical for `n` up to about 20.

---

## 7. Traps

The classic precedence bug:
```
6 & 1 == 0        -> 0 (false)   wrong: parsed as 6 & (1 == 0)
(6 & 1) == 0      -> 1 (true)    correct
```

- **Precedence:** `==` binds tighter than `&`, `^` and `|`. That is the opposite of what most people expect, so `n & 1 == 0` is `n & (1 == 0)`, which is always 0.
- **Brackets:** whenever you mix bit operators with arithmetic or comparisons, add brackets. This is the rule for interviews.
- **`cout`:** `cout << a & b` does not compile, because `<<` binds tighter than `&`. Write `cout << (a & b)`.
- **Overflow:** `1 << k` is an `int`. For `k` of 31 or more, use `1LL << k`.
- **Shift count:** `k` must be less than the bit width (32 for an `int`). Shifting by the width of the type or more (`1 << 32` on an `int`) is undefined.
- **Signed numbers:** do not left-shift negative numbers, and remember that `>>` on a negative number rounds down (see `>>` in section 1). Use `unsigned` types for bit work.
- **Negative `n` in base 2:** `n % 2` is not `n & 1`, and `n / 2` is not `n >> 1`. `-3 % 2` is `-1` but `-3 & 1` is `1`, and `-3 / 2` is `-1` but `-3 >> 1` is `-2`. The swaps in section 8 hold only for `n >= 0`.
- **XOR swap on one variable:** if `a` and `b` are the same variable, for example `v[i]` and `v[j]` with `i == j`, the first step `a ^= b` is `x ^ x`, which is 0, and the value is lost. With `v = {7, 9}`, XOR-swapping `v[0]` with `v[0]` leaves `v[0] = 0`. Two different variables with the same value are fine: `a = 5, b = 5` stays `5, 5`. Check `i != j` first, or use `std::swap`.

---

## 8. Number bases and binary

For what a base is, how to convert a number to any base, and the general `toBase` / `fromBase` helpers (bases 7, 16, 64), see "Number bases" in `Notes_strings.md`. This section covers base 2.

Base 2 positions are worth 1, 2, 4, 8, 16, ..., which are the bits.

100 to base 2, dividing by 2 each time:
```
100 -> 50 r0 -> 25 r0 -> 12 r1 -> 6 r0 -> 3 r0 -> 1 r1 -> 0 r1
read the remainders right to left: "1100100"
```

**For base 2** (and `n >= 0`; for a negative `n` they differ, see section 7), divide and remainder are bit operations:
```
n % 2    is   n & 1     (read the last bit)
n / 2    is   n >> 1    (remove the last bit)
```

Converting `n >= 0` to a binary string:
```cpp
string toBinary(int n) {                         // n >= 0
    if (n == 0) return "0";
    string s;
    while (n > 0) {
        s.push_back('0' + (n & 1));              // last bit as '0' or '1'
        n >>= 1;
    }
    reverse(s.begin(), s.end());
    return s;
}
```
```
toBinary(100)                    -> "1100100"
toBinary(5)                      -> "101"
toBinary(0)                      -> "0"
bitset<8>(100)                   -> 01100100      (fixed width, header <bitset>)
bitset<8>(100).to_string()       -> "01100100"
stoi("1100100", nullptr, 2)      -> 100           (binary string back to a number)
```

- For a negative `n`, convert `(unsigned)n` to get its two's complement bits.

**Practice:** 67 Add Binary, 504 Base 7, 405 Convert a Number to Hexadecimal, 191 Number of 1 Bits.
