# Math Notes (DSA interviews)

## 1. Modulo (`%`)

**`a % k`:** the remainder after dividing `a` by `k`.

**`a ≡ b (mod k)`:** `a` and `b` have the same remainder when divided by `k`. This holds exactly when `a - b` is divisible by `k`.

### Negative remainders

**In maths, a remainder is always between `0` and `k-1`.** `-7 mod 5` is `3`, because `-7 = 5 × (-2) + 3`.

**C++ gives a negative remainder when the left number is negative.** C++ division rounds toward zero, so `%` keeps the sign of the left number:
```
-7 % 5     -> -2      (maths: 3)
 7 % 5     ->  2
-10 % 5    ->  0
```

**Why convert to positive:** `-2` and `3` are the same remainder for `k = 5` (they differ by 5), but C++ treats them as different values.
- As map keys they never match, so counts are missed.
- As array indexes, `seen[-2]` is out of bounds.

**Fix: add `k`, the mod number.** A C++ remainder is never below `-(k-1)`, so adding `k` makes it positive and gives the maths answer. The second `% k` brings a remainder that was already positive back below `k`.
```cpp
int r = ((a % k) + k) % k;      // always 0..k-1
```
```
((-7 % 5) + 5) % 5   -> 3     (-2 + 5 = 3)
(( 7 % 5) + 5) % 5   -> 2     (2 + 5 = 7, then 7 % 5 = 2)
```

- Needed only when `a` can be negative: negative input, or after a subtraction.

### Formulas

In the first four lines, the left side is the maths remainder (`mod`, always `0..k-1`) and the right side is how to get it with C++ `%`. All six lines assume `a, b >= 0`, so normalise a negative value first (above).
```
(a + b) mod k     ==  ((a % k) + (b % k)) % k
(a - b) mod k     ==  ((a % k) - (b % k) + k) % k      (+ k because the middle can go negative)
(a * b) mod k     ==  ((a % k) * (b % k)) % k          (multiply in long long)
(a + m * k) mod k ==  a % k                            (adding a multiple of k changes nothing)

(a - b) % k == 0   <=>   a % k == b % k              (same remainder: difference divisible)
(a + b) % k == 0   <=>   a % k == (k - b % k) % k    (outer % handles b % k == 0)
```

- **Trap:** the `%` lines break on a negative value. `-1 % 5 == 4 % 5` is false, although `-1 - 4` is divisible by 5 (the bug in 974). And `(2 - 5) % 4` is `-3`, while the `(a - b)` line gives the maths answer: `((2 % 4) - (5 % 4) + 4) % 4` is `1`.
- Division does not work: `(12 / 4) % 5` is `3`, but `(12 % 5) / (4 % 5)` is `0`.

### Which remainder to look for

**Sum of two values: look for the opposite remainder.** You need `a + b ≡ 0 (mod k)`, so the partner's remainder must be `-r mod k` (where `r = a % k`), which is `(k - r) % k`.
- From `a + b ≡ 0`, move `a` to the other side: `b ≡ -a ≡ -r`.
- `-r` is negative, so add `k`: `k - r` is the same remainder (adding `k` changes nothing).
- For `r = 0`, `k - r` is `k`, which is not a valid remainder (`0..k-1`). The outer `% k` turns it into `0`, so remainder `0` pairs only with remainder `0`.

With `k = 60`:
```
k = 60, r = 20   -> partner 40   (20 + 40 = 60)
k = 60, r = 30   -> partner 30   (30 + 30 = 60)
k = 60, r = 0    -> partner 0    ((60 - 0) % 60 = 0, not 60)
```

**Difference of two values: look for the same remainder.** `a - b ≡ 0 (mod k)` means `b ≡ a`, so the partner's remainder is `r` itself. Nothing wraps, so `r = 0` needs no special case. Subarray sums use this, because `sum(j..i) = prefix[i] - prefix[j-1]`.

| Problem says                   | Use                                              |
|--------------------------------|--------------------------------------------------|
| subarray sum divisible by `k`  | `prefix[j-1] % k == prefix[i] % k` (974, 523)    |
| pair sum divisible by `k`      | `nums[j] % k == (k - nums[i] % k) % k` (1010)    |
| return the answer modulo 10⁹+7 | `%` after every `+` and `*`, `long long` for `*` |

- With 10⁹+7, also use the `(a - b)` formula above after every `-`, or the result can go negative.

### Why `long long` is enough for one `*`

Both remainders are below 10⁹+7, so their product is below about 1.0 × 10¹⁸. `long long` holds up to about 9.2 × 10¹⁸, so one product fits with room to spare. `int` holds only about 2.1 × 10⁹, so the product does not fit.

The largest product, checked:
```
(10⁹+6) × (10⁹+6)   -> 1,000,000,012,000,000,036     (about 1.0 × 10¹⁸)
LLONG_MAX           -> 9,223,372,036,854,775,807     (about 9.2 × 10¹⁸)
```

It is not enough when:
- **You multiply three values before the `%`:** `a * b * c` can reach 10²⁷. Take `%` after each `*`: `(a * b % MOD) * c % MOD`.
- **The values were never reduced:** `%` each one first, so it is below the modulus.
- **The modulus is above about 3 × 10⁹:** the square of the largest remainder no longer fits. Use `__int128` for the product.
- **Both values are `int`:** `long long x = a * b;` multiplies in `int` and overflows before the result is converted. Convert one side first: `(long long)a * b`, or put `1LL` in front: `1LL * a * b` (see section 2).

**Practice:** 1010 Pairs of Songs With Total Durations Divisible by 60, 974 Subarray Sums Divisible by K, 523 Continuous Subarray Sum.

---

## 2. Overflow and `long long`

An `int` is 4 bytes (32 bits) and holds up to 2³¹ - 1. Past that it overflows, which is undefined behaviour for a signed type. When a sum or a product can pass that, do the arithmetic in `long long`.

A signed type of `b` bits goes from -2^(b-1) to 2^(b-1) - 1, because one bit is the sign. The `unsigned` version of a type doubles the positive range (x2), because that bit holds a value instead.

Limits, from `<climits>`:
```
INT_MAX      =  2³¹ - 1   ->   2,147,483,647
INT_MIN      = -2³¹       ->  -2,147,483,648
LLONG_MAX    =  2⁶³ - 1   ->   9,223,372,036,854,775,807
LLONG_MIN    = -2⁶³       ->  -9,223,372,036,854,775,808
```

- No standard signed type is bigger than `long long`. `__int128` (GCC and clang only, up to 2¹²⁷ - 1) is the next step. `cout << x` does not compile for it, so convert it to a string to print it.

### The `LL` suffix

**Suffix:** letters written after a number that set its type. A plain `1` is an `int`, so `1LL` is how you write a `long long`.

Type of each literal:
```
1       -> int                  (4 bytes)
1LL     -> long long            (8 bytes)
1ULL    -> unsigned long long   (8 bytes)
```

- `ll` and `LL` mean the same, so `1ll` and `1LL` are identical to the compiler. Mixed case (`1lL`, `1Ll`) does not compile.
- **Prefer `LL`.** A lowercase `l` looks like `1` in many fonts, so `1ll` can read as `11`.

### One `long long` operand makes the product `long long`

Each `*` looks only at its own two operands. If one of them is `long long`, the other is converted to `long long` and the multiplication runs in `long long`. The type of the variable that stores the result does not change how the multiplication runs.

`*` goes left to right, so the `1LL` must come before any `int * int` part. With `int a = 100000, b = 100000` (the true product is 10¹⁰):
```cpp
long long x = 1LL * a * b;        // 10000000000   (1LL * a) * b, both in long long
long long y = a * (1LL * b);      // 10000000000
long long z = a * b * 1LL;        // 1410065408    (a * b) runs in int and overflows first
long long w = a * b;              // 1410065408    same overflow
long long v = (long long)a * b;   // 10000000000   a cast works too
```

- The wrong values are what typical compilers give. The standard leaves the result undefined.
- The same idea applies to shifts: `1LL << k`. See the "Overflow" trap in section 7 of `Notes_bits.md`.

> [!WARNING]
> `long long x = a * b;` still overflows when `a` and `b` are `int`. Put `1LL *` in front, or cast one side, before the multiplication.

**Practice:** 7 Reverse Integer, 8 String to Integer (atoi).
