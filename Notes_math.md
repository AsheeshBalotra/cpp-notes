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

> TODO: Expand — why `long long` is enough for one `*` of two remainders below 10⁹+7, and when it is not.

**Practice:** 1010 Pairs of Songs With Total Durations Divisible by 60, 974 Subarray Sums Divisible by K, 523 Continuous Subarray Sum.
