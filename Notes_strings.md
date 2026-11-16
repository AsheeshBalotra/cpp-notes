# String Notes (DSA interviews)

**Abbreviations**
- **ASCII:** American Standard Code for Information Interchange

## 1. Characters: ASCII and `char` arithmetic

### ASCII values to remember

ASCII (American Standard Code for Information Interchange) gives each character a number:
```
'0' = 48    '9' = 57
'A' = 65    'Z' = 90
'a' = 97    'z' = 122
```

- Order is digits, then uppercase, then lowercase.
- `'a' - 'A'` is 32.

### Why 32 works

Uppercase and lowercase letters differ only in one bit, the value 32. That is why `| 32`, `& ~32` and `^ 32` all work.
```
'A' = 65 = 0100 0001
'a' = 97 = 0110 0001
             ^ bit 5 (value 32) is the only difference

'A' | 32    -> 'a'    (set bit 5: to lowercase)
'a' | 32    -> 'a'    (already set, no change)
'a' & ~32   -> 'A'    (clear bit 5: to uppercase)
'A' & ~32   -> 'A'    (already clear, no change)
'A' ^ 32    -> 'a'    (flip bit 5: toggles the case)
'a' ^ 32    -> 'A'
```

- For what `|`, `&`, `^` and `~` do, see section 1 of `Notes_bits.md`.
- The result is an `int`, so cast it: `(char)(c | 32)`.
- Use these only on letters. A digit or symbol can turn into something else.

### `char` arithmetic gives `int`

Results and their types:
```
1 + 'a'           -> 98    (int)
'a' + 'b'         -> 195   (int)
'a'               -> a     (no arithmetic, stays char)
'a' + 1           -> 98
(char)('a' + 1)   -> b
'd' - 'a'         -> 3
'7' - '0'         -> 7
```

- `char`, `short` and `bool` are promoted to `int` before any arithmetic.
- Small types are widened to `int` because `int` is the size the CPU computes with.
- "Smaller than `int`" means fewer bytes. Mixed types convert up: `int + double` is `double`.
- `c += 1` and `c++` keep a `char` a `char`. `c + 1` is an `int`.

| Type        | Size    | Range              |
|-------------|---------|--------------------|
| `bool`      | 1 byte  | 0 or 1             |
| `char`      | 1 byte  | -128 to 127        |
| `short`     | 2 bytes | -32,768 to 32,767  |
| `int`       | 4 bytes | about ±2.1 billion |
| `long long` | 8 bytes | about ±9.2 × 10¹⁸  |

---

## 2. `stoi` (string to int)

Header `<string>`. Range is -2147483648 to 2147483647. For bigger values use `stoll`.

What `stoi` returns for each input:
```
stoi("42")           -> 42
stoi("-42")          -> -42
stoi("+42")          -> 42
stoi("007")          -> 7
stoi("-007")         -> -7
stoi("-0")           -> 0
stoi("   -42")       -> -42
stoi("42   ")        -> 42
stoi("42abc")        -> 42
stoi("3.99")         -> 3
stoi("4 2")          -> 4
stoi("abc")          -> throws invalid_argument
stoi("--5")          -> throws invalid_argument
stoi("")             -> throws invalid_argument
stoi("2147483648")   -> throws out_of_range
```

- It needs a digit at the start (after optional spaces and one `+` or `-`).
- It stops quietly at the first non-digit.

---

## 3. `to_string` (number to string)

Header `<string>`. The reverse of `stoi`.
- A `double` always gets 6 decimal places.
- A negative number keeps its `-`, so `to_string(-56).size()` is 3.

What `to_string` returns:
```
to_string(1234)                       -> "1234"
to_string(-56)                        -> "-56"
to_string(3.14)                       -> "3.140000"
to_string(12) + "," + to_string(34)   -> "12,34"
to_string(1234).size()                -> 4
```

Adding up the digits of a number:
```cpp
for (char ch : to_string(1234)) sum += ch - '0';   // sum = 10
```

---

## 4. `substr` (substring)

`s.substr(pos, len)` returns a copy. **`len` is a length, not an end index.**
```
s = "Hello, World!"

s.substr(7, 5)          -> "World"
s.substr(7)             -> "World!"
s.substr(0, 5)          -> "Hello"
s.substr(s.size() - 3)  -> "ld!"
s.substr(7, 100)        -> "World!"
s.substr(s.size())      -> ""
s.substr(s.size() + 1)  -> throws out_of_range
s.substr(i, j - i + 1)  -> index i to j inclusive (i = 7, j = 11 gives "World")
```

Split at a delimiter: find where the separator is, then cut before it and after it.
```
p = "name=Ada",  eq = p.find('=')
p.substr(0, eq)         -> "name"
p.substr(eq + 1)        -> "Ada"
```

> TODO: Expand — what the split does when the delimiter is missing (`find` returns `npos`).

---

## 5. `find` / `rfind` (search)

`s.find(x, pos)` finds the first match, and `s.rfind(x, pos)` finds the last. `x` is a string or a `char`. `pos` is optional: where to start (for `rfind`, the latest index a match may start at).
They return the index of the match, or `string::npos` if there is none.
```
s = "the cat sat on the mat"

s.find("cat")        -> 4
s.find('s')          -> 8
s.find("the", 1)     -> 15
s.rfind("the")       -> 15
s.rfind("the", 14)   -> 0
s.find("dog")        -> npos
```

Checking for a match, and looping over every match:
```cpp
if (s.find("cat") != string::npos) { /* contains */ }

for (size_t p = s.find("at"); p != string::npos; p = s.find("at", p + 1))
    cout << p << " ";            // 5 9 20
```

- `npos` is the largest `size_t` value, so compare against it.
- **Trap:** `if (s.find("cat"))` is wrong. It is false when the text is found at index 0 and true when it is not found.

---

## 6. `erase` (remove characters)

It changes `s` itself.
- `erase(pos, len)`: removes `len` characters from `pos`. `len` is a length, not an end index, and is optional (the default is to the end).
- `erase(it)`: removes one character.
- `erase(first, last)`: removes `[first, last)`.

The first form returns `s`. The iterator forms return an iterator to the next character.

What each call leaves in `s`:
```
s = "Hello World"                       (each line starts again from this s)

s.erase(5, 1)                       -> "HelloWorld"
s.erase(5)                          -> "Hello"
s.erase(5, 100)                     -> "Hello"
s.erase(s.size() - 1)               -> "Hello Worl"
s.erase(s.begin() + 5)              -> "HelloWorld"
s.erase(s.begin(), s.begin() + 6)   -> "World"
s.erase()                           -> ""
s.erase(s.size() + 1)               -> throws out_of_range
```

> TODO: Expand — the time cost of one `erase` call, and what it costs to call `erase` inside a loop.

---

## 7. Case helpers: `tolower` / `toupper`, `isupper` / `islower`

All four work on one character (header `<cctype>`) and return an `int`.
```
(char)tolower('Q')   -> 'q'
(char)toupper('q')   -> 'Q'
isupper('A')         -> non-zero (true)
islower('A')         -> 0 (false)
```

- **Trap:** `isupper(c) == true` is wrong. The `is...` functions return non-zero for true, not always 1, and `true` converts to 1, so the comparison can fail. Write `if (isupper(c))`.

Whole string (C++ has no built-in):
```cpp
string toLower(string s) {
    transform(s.begin(), s.end(), s.begin(), [](unsigned char c) { return (char)tolower(c); });
    return s;
}
// toUpper: the same with toupper
```
```
toLower("Hello World")   -> "hello world"
toUpper("Hello World")   -> "HELLO WORLD"
```

- Cast the result to `char`: `cout << tolower('Q')` prints `113`, not `q`.

---

## 8. Number bases (base 2, 7, 16, 64)

**What a base means:** base `b` uses the digits `0` to `b - 1`, and each position is worth `b` times the one to its right.
```
base 10:  345      = 3×100 + 4×10 + 5×1
base 7:   202      = 2×49  + 0×7  + 2×1          = 100
base 2:   1100100  = 64 + 32 + 4                  = 100
base 16:  64       = 6×16  + 4×1                  = 100
```

100 (base 10), 202 (base 7), 1100100 (base 2) and 64 (base 16) are the same number written in different systems.

**Number to base `b`:** take `n % b` as the next digit (from the right), set `n = n / b`, repeat until `n` is 0, then reverse.
```
100 to base 7:
100 / 7 = 14   remainder 2    <- last digit
 14 / 7 =  2   remainder 0
  2 / 7 =  0   remainder 2    <- first digit
read upward: "202"
```

Any base from 2 to 36:
```cpp
string toBase(int n, int base) {                 // base 2 to 36
    if (n == 0) return "0";
    const string digits = "0123456789abcdefghijklmnopqrstuvwxyz";
    long long x = n;                             // long long so -INT_MIN does not overflow
    bool negative = x < 0;
    if (negative) x = -x;
    string s;
    while (x > 0) {
        s.push_back(digits[x % base]);           // remainder picks the digit symbol
        x /= base;
    }
    if (negative) s.push_back('-');
    reverse(s.begin(), s.end());
    return s;
}
```
```
toBase(100, 7)    -> "202"
toBase(100, 2)    -> "1100100"
toBase(100, 16)   -> "64"
toBase(255, 16)   -> "ff"
toBase(-7, 7)     -> "-10"
toBase(0, 7)      -> "0"
```

**Base `b` string to number:** read left to right, `n = n * b + digit`.
```cpp
int fromBase(const string& s, int base) {
    int n = 0;
    for (char c : s) n = n * base + (c - '0');   // digits 0-9 only
    return n;
}
```
```
fromBase("202", 7)         -> 100
fromBase("1100100", 2)     -> 100
stoi("202", nullptr, 7)    -> 100     (stoi does this for bases 2 to 36)
stoi("ff", nullptr, 16)    -> 255
```

**Base 64** needs 64 symbols. The standard alphabet is `A-Z` = 0-25, `a-z` = 26-51, `0-9` = 52-61, `+` = 62, `/` = 63.
```
100 / 64 = 1 remainder 36   -> digits 1, 36   -> "Bk"
```

- "Base64 encoding" (emails, URLs, images) is different: it turns raw bytes into text 6 bits at a time, using the same alphabet.

Remember:
- Digits come out last first, so `push_back` then `reverse` once (same as Add Strings).
- `0` needs its own check, or the loop never runs and you return `""`.
- Negatives: convert the absolute value, then add `-`.
- Bases above 10 use letters for digits: in base 16, `a` = 10 and `f` = 15.

**Practice:** 504 Base 7, 67 Add Binary, 405 Convert a Number to Hexadecimal, 168 Excel Sheet Column Title (base 26 with no zero digit, so subtract 1 before each `%`), 171 Excel Sheet Column Number.
