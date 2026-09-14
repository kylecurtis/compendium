# Integral Types

Integral types are fundamental C++ types whose values are discrete rather than fractional. They include **boolean types**, **character types**, and the standard **signed and unsigned integer types**.

Understanding integral types requires more than memorizing `int`, `long`, and `unsigned`. Correct and portable C++ also depends on understanding **range guarantees, signedness, integer literals, promotions, conversions, overflow, size-related types, fixed-width types, and platform data models**.

<br />

---

<br />

## Integral Type Categories

The fundamental integral types can be viewed as three related groups:

```text
Integral types
├── Boolean type
│   └── bool
│
├── Character types
│   ├── char
│   ├── signed char
│   ├── unsigned char
│   ├── wchar_t
│   ├── char8_t   // C++20
│   ├── char16_t  // C++11
│   └── char32_t  // C++11
│
└── Standard integer types
    ├── signed char
    ├── short
    ├── int
    ├── long
    ├── long long
    └── corresponding unsigned types
```

> [!important]
> `char`, `signed char`, and `unsigned char` are **three distinct types**. Plain `char` is not simply shorthand for one of the other two.

<br />

---

<br />

## `bool`

`bool` is the built-in boolean type and has exactly two values:

```cpp
true
false
```

Prefer boolean literals when initializing boolean values:

```cpp
bool is_ready{true};
bool has_error{false};
```

When converted to an integer:

```text
false -> 0
true  -> 1
```

When an arithmetic, enumeration, pointer, or pointer-to-member value is converted to `bool`, zero/null becomes `false` and non-zero/non-null becomes `true`.

```cpp
bool a{0};  // false
bool b{1};  // true

bool c = 42; // true
```

> [!important]
> Do not infer the object representation of `bool` from the fact that conversions produce `0` and `1`. The language guarantees its two values and conversion behavior; `sizeof(bool)` is implementation-defined.

<br />

---

<br />

## Standard Integer Types

The ordinary signed integer types are:

```cpp
signed char
short
int
long
long long
```

The corresponding unsigned types are:

```cpp
unsigned char
unsigned short
unsigned int
unsigned long
unsigned long long
```

The keyword `int` may be omitted when another integer modifier makes the type unambiguous:

```cpp
short int a{};
short b{};              // same type

long int c{};
long d{};               // same type

unsigned int e{};
unsigned f{};           // same type

long long int g{};
long long h{};          // same type
```

For the ordinary integer families, signedness defaults to `signed` when `unsigned` is not written:

```cpp
int x{};         // signed int
signed int y{};  // same type

long z{};        // signed long
```

> [!important]
> This rule does **not** mean plain `char` is always signed. Plain `char` has implementation-defined signedness behavior and is a distinct type.

<br />

---

<br />

## `char`, `signed char`, and `unsigned char`

These are distinct fundamental types:

```cpp
char
signed char
unsigned char
```

`unsigned char` is unsigned and `signed char` is signed.

Plain `char` is different: whether its range behaves like `signed char` or `unsigned char` is implementation-defined.

```cpp
char c{'A'};
signed char s{100};
unsigned char u{200};
```

Code that assumes plain `char` can represent negative values is therefore not portable.

<br />

### `char` and Bytes

C++ defines one byte as the size of `char`:

```cpp
sizeof(char) == 1
sizeof(signed char) == 1
sizeof(unsigned char) == 1
```

However, a C++ byte is **not definitionally 8 bits**.

The number of bits in one byte is given by `CHAR_BIT` from `<climits>`:

```cpp
#include <climits>

static_assert(CHAR_BIT >= 8);
```

On nearly all mainstream desktop and server systems today:

```text
CHAR_BIT == 8
```

but portable language rules allow larger values.

Therefore:

```cpp
sizeof(T) * 8
```

is not a fully portable way to determine the number of bits occupied by `T`.

The portable expression is:

```cpp
sizeof(T) * CHAR_BIT
```

<br />

### Character-Oriented Integral Types

C++ also provides:

```cpp
wchar_t
char8_t   // C++20
char16_t  // C++11
char32_t  // C++11
```

These are distinct fundamental types intended primarily for character/code-unit representation rather than ordinary arithmetic.

Although they are integral types, their semantic role is different enough that they should not normally be chosen merely because their width happens to suit an integer calculation.

<br />

---

<br />

## What C++ Guarantees About Integer Sizes

The standard deliberately does not say that `int` is always 32 bits or that `long` is always 64 bits.

Instead, it provides minimum width/range guarantees and an ordering relationship.

For the standard integer types:

```cpp
sizeof(char)
    <= sizeof(short)
    <= sizeof(int)
    <= sizeof(long)
    <= sizeof(long long)
```

The standard minimum widths are:

| Type family | Minimum width |
|---|---:|
| `char`, `signed char`, `unsigned char` | 8 bits |
| `short`, `unsigned short` | 16 bits |
| `int`, `unsigned int` | 16 bits |
| `long`, `unsigned long` | 32 bits |
| `long long`, `unsigned long long` | 64 bits |

> [!important]
> These are **minimum guarantees**, not universal exact widths. An implementation may provide wider types.

<br />

---

<br />

## C++20 and Later: Signed Integer Representation

Since C++20, the standard signed integer types use **two's-complement representation**.

For an `N`-bit signed integer representation, the familiar range is:

```text
-2^(N - 1) through 2^(N - 1) - 1
```

For an `N`-bit unsigned integer representation:

```text
0 through 2^N - 1
```

Using the standard minimum widths, C++20+ therefore guarantees at least:

| C++ type | Minimum guaranteed value | Maximum guaranteed value |
|---|---:|:---|
| `signed char` | -128 | 127 |
| `unsigned char` | 0 | 255 |
| `short` | -32,768 | 32,767 |
| `unsigned short` | 0 | 65,535 |
| `int` | -32,768 | 32,767 |
| `unsigned int` | 0 | 65,535 |
| `long` | -2,147,483,648 | 2,147,483,647 |
| `unsigned long` | 0 | 4,294,967,295 |
| `long long` | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807 |
| `unsigned long long` | 0 | 18,446,744,073,709,551,615 |

A useful signed-range identity is:

```cpp
MIN == -MAX - 1
```

For example, for an implementation's `int` limits:

```cpp
INT_MIN == -INT_MAX - 1
```

<br />

### Historical Note: Before C++20

Before C++20, the standard permitted signed integer representations including:

```text
two's complement
one's complement
sign-and-magnitude
```

Because one's-complement and sign-and-magnitude representations can contain both positive and negative zero, the portable minimum range for an `N`-bit signed type was historically:

```text
-(2^(N - 1) - 1) through +(2^(N - 1) - 1)
```

For example, the historical portable lower guarantee corresponding to a 32-bit signed representation was:

```text
-2,147,483,647
```

rather than:

```text
-2,147,483,648
```

Mainstream implementations had already used two's complement for many years; C++20 aligned the language requirement with that reality.

> [!info]
> This history matters when reading older standards, books, numeric-limit tables, or portability discussions, but new C++20+ code can rely on two's-complement representation for the standard signed integer types.

<br />

---

<br />

## Signed vs. Unsigned Arithmetic

Signed and unsigned integers do not merely differ in range. Their arithmetic has different overflow semantics.

<br />

### Signed Overflow

Overflow of a signed integer operation is **undefined behavior**.

```cpp
#include <climits>

int x{INT_MAX};
++x; // undefined behavior
```

Two's-complement representation does **not** make signed overflow wraparound behavior defined.

> [!important]
> Never intentionally rely on signed integer overflow wrapping from the maximum value to the minimum value.

<br />

### Unsigned Arithmetic

Unsigned arithmetic is defined modulo `2^N`, where `N` is the number of value bits of the unsigned type.

For example, on a system with a 32-bit `unsigned int`:

```cpp
unsigned int x{0};
--x;
```

produces:

```text
4,294,967,295
```

The wraparound is defined behavior.

This is useful for operations where modular arithmetic is intentional, but it can be surprising when unsigned types are used merely to express "this value should never be negative."

<br />

### Choosing Signed or Unsigned Types

For ordinary arithmetic, counts, loop variables, and values where negative intermediate results may be meaningful, a signed type such as `int` is often the clearest default when its range is sufficient.

Do not choose an unsigned type solely because a logical quantity should never be negative.

Use unsigned types when their semantics are appropriate, such as:

- intentional modular arithmetic;
- bit manipulation;
- APIs or binary interfaces that require an unsigned type;
- size-related APIs that use `std::size_t`;
- exact-width unsigned storage such as `std::uint32_t` when width is part of the contract.

> [!important]
> Mixing signed and unsigned arithmetic is a common source of bugs because the signed operand may be converted to an unsigned type before the operation is performed.

<br />

---

<br />

## Integral Promotions

Before many arithmetic operations, integral values narrower than `int` are promoted.

Types such as:

```cpp
bool
char
signed char
unsigned char
short
unsigned short
```

undergo **integral promotion** when used in many expressions.

If `int` can represent every value of the source type, the value is promoted to `int`. Otherwise, it is promoted to `unsigned int`.

On mainstream systems this means operations on `char` and `short` usually happen as `int` arithmetic:

```cpp
short a{10};
short b{20};

auto result{a + b};
```

`result` is normally `int`, not `short`.

```cpp
static_assert(std::is_same_v<decltype(result), int>);
```

assuming the ordinary platform relationship where `int` can represent every `short` value.

<br />

### `bool` Promotion

`bool` promotes to `int`:

```cpp
bool flag{true};
auto x{flag + 4}; // int, value 5
```

This is legal but is rarely good expressive code outside specialized situations.

<br />

---

<br />

## Integer Conversion Rank

Each standard integer type has a **conversion rank** used by integral promotions and the usual arithmetic conversions.

For the signed standard integer types, rank increases in this order:

```text
signed char
    < short
    < int
    < long
    < long long
```

The unsigned counterpart of a signed integer type has the same conversion rank as that signed type.

For example:

```text
rank(int) == rank(unsigned int)
rank(long) == rank(unsigned long)
```

No two signed standard integer types have the same rank.

> [!info]
> Conversion rank is a language rule used to decide common arithmetic types. It is not simply another name for `sizeof(T)`.

<br />

---

<br />

## Usual Arithmetic Conversions for Integers

When binary arithmetic or comparison operators receive different arithmetic types, C++ determines a common type before carrying out the operation.

For integral operands, the process conceptually begins by applying integral promotions.

After promotion, if both operands have the same type, no further conversion is needed.

If both are signed or both are unsigned, the lower-rank type is converted to the higher-rank type.

```cpp
int a{10};
long b{20};

auto result{a + b}; // typically long
```

The more surprising cases happen when one operand is signed and the other is unsigned.

<br />

### Mixed Signed and Unsigned Operands

Consider:

```cpp
int x{-1};
unsigned int y{1};

bool result{x < y};
```

On ordinary implementations where `int` and `unsigned int` have the same rank and `int` cannot represent all values of `unsigned int`, `x` is converted to `unsigned int`.

The converted `-1` becomes a large unsigned value, making the comparison:

```cpp
x < y
```

evaluate to `false`.

This is why code should avoid casual mixtures of signed and unsigned arithmetic.

<br />

### Simplified Signed/Unsigned Decision Rules

After integral promotions, for one signed type `S` and one unsigned type `U`:

1. If `U` has rank greater than or equal to `S`, `S` converts to `U`.
2. Otherwise, if `S` can represent every value of `U`, `U` converts to `S`.
3. Otherwise, both convert to the unsigned type corresponding to `S`.

These rules are designed to preserve representable values where possible, but they can still produce unintuitive results when negative signed values meet unsigned operands.

<br />

### Safer Mixed-Signedness Comparisons in C++20

C++20 provides integer comparison helpers in `<utility>`:

```cpp
#include <utility>

std::cmp_equal
std::cmp_not_equal
std::cmp_less
std::cmp_greater
std::cmp_less_equal
std::cmp_greater_equal
```

They are useful when comparing integer values of different signedness without allowing the ordinary conversion rules to silently reinterpret a negative signed value as a large unsigned value.

```cpp
int x{-1};
unsigned int y{1};

bool a{x < y};              // false on ordinary implementations
bool b{std::cmp_less(x, y)}; // true
```

<br />

---

<br />

## Conversions Between Integral Types

Integral conversions occur when a value is converted to another integral type after any relevant promotion rules.

<br />

### Conversion to an Unsigned Type

Conversion to an unsigned integer type is well-defined modulo one more than the maximum value representable by the destination type.

For an `N`-bit unsigned destination:

```text
result is congruent to the source value modulo 2^N
```

Example on a typical 32-bit `unsigned int`:

```cpp
int x{-1};
unsigned int y{x};
```

produces:

```text
4,294,967,295
```

<br />

### Conversion to a Signed Type

If the source value is representable in the destination signed type, the value is preserved.

Since C++20, if the source value is not representable, the result is the unique value of the destination type congruent to the source value modulo `2^N`, where `N` is the number of bits used to represent the destination type.

```cpp
long long large{5'000'000'000LL};
int x{static_cast<int>(large)}; // C++20+: modulo-based integral conversion
```

Before C++20, an out-of-range conversion to a signed integer type produced an implementation-defined result.

> [!important]
> This is different from signed **arithmetic overflow**, which remains undefined behavior. Integral conversion and arithmetic overflow are separate language rules.

<br />

### Narrowing and Brace Initialization

List initialization rejects many narrowing integral conversions at compile time:

```cpp
long long large{5'000'000'000LL};

int a = large; // allowed; may lose information
int b{large};  // error: narrowing conversion
```

This is one reason brace initialization is useful when creating integral variables.

<br />

---

<br />

## Integer Literals

Integer literals are integer values written directly in source code.

They can be written in several numeric bases and can optionally use suffixes that affect the literal's type.

<br />

### Decimal Literals

A decimal literal uses base 10.

```cpp
int decimal{42};
```

An ordinary non-zero decimal literal begins with a digit from `1` through `9` and may be followed by additional decimal digits.

```cpp
42
1000
987654
```

Zero itself is written as:

```cpp
0
```

<br />

### Octal Literals

An integer literal beginning with `0` and followed by digits `0` through `7` is octal.

```cpp
int octal{052}; // decimal 42
```

> [!warning]
> Leading zeroes can accidentally change the numeric base.
>
> ```cpp
> int x{010}; // decimal value 8, not 10
> ```

<br />

### Hexadecimal Literals

Hexadecimal literals begin with `0x` or `0X`:

```cpp
int a{0x2A};
int b{0X2a};
```

Both represent decimal `42`.

Hexadecimal digits are:

```text
0-9
A-F
a-f
```

<br />

### Binary Literals (C++14)

Binary literals begin with `0b` or `0B`:

```cpp
int a{0b101010};
int b{0B101010};
```

Both represent decimal `42`.

Binary literals are particularly useful for masks and bit-oriented code.

<br />

### Digit Separators (C++14)

Single quotes may separate digits for readability:

```cpp
int population{1'000'000};
unsigned mask{0b1111'0000U};
auto hex{0xFFFF'0000U};
```

The separators do not affect the value or type of the literal.

<br />

---

<br />

## Integer Literal Types and Suffixes

A literal suffix controls the **type-selection rules for the literal itself**.

It does not need to match the type of the variable receiving the literal.

For example:

```cpp
long a{32};       // valid: literal 32 is initially int
long long b{32};  // valid
```

Common suffixes include:

| Suffix | Meaning |
|---|---|
| `U` / `u` | unsigned |
| `L` / `l` | long |
| `LL` / `ll` | long long |
| `Z` / `z` | size suffix (C++23) |

Suffix components may be combined where permitted:

```cpp
32U
32L
32UL
32LL
32ULL
```

Uppercase `L` is usually preferred over lowercase `l` because lowercase `l` can resemble the digit `1`.

<br />

### Unsuffixed Decimal Literal Type Selection

For an unsuffixed decimal literal, C++ chooses the first type that can represent its value from:

```text
int
long
long long
```

Example:

```cpp
auto x{42}; // int
```

A sufficiently large decimal value may therefore become `long` or `long long` without an explicit suffix.

<br />

### Unsuffixed Binary, Octal, and Hexadecimal Literals

For unsuffixed non-decimal literals, the candidate sequence additionally includes unsigned types:

```text
int
unsigned int
long
unsigned long
long long
unsigned long long
```

This means a hexadecimal literal and a decimal literal with the same mathematical value can potentially receive different types when the value is sufficiently large.

> [!important]
> Numeric base can affect not only readability, but also **literal type selection**.

<br />

### `U` Suffix

For a literal with a `U` suffix, the candidate types are unsigned:

```cpp
auto x{42U}; // unsigned int
```

For sufficiently large values, a wider unsigned type may be selected according to the literal's base and suffix rules.

<br />

### `L` and `LL` Suffixes

```cpp
auto a{42L};  // long

auto b{42LL}; // long long
```

provided the value is representable in the requested type.

Unsigned combinations are also available:

```cpp
auto a{42UL};
auto b{42ULL};
```

<br />

### Size Suffix `Z` / `z` (C++23)

C++23 adds a size suffix associated with `std::size_t` and its signed counterpart.

```cpp
#include <cstddef>

auto a{42uz}; // std::size_t
```

A `z` suffix without `u` selects the signed version of `std::size_t` according to the integer-literal rules.

This can be useful when a literal is intended to participate naturally in size-oriented code.

<br />

### Negative Integer Literals Are Unary Expressions

The minus sign is not part of the integer literal token.

For example:

```cpp
-42
```

is conceptually:

```text
unary minus applied to the literal 42
```

This matters for extreme negative values because the positive literal must first have a valid type before unary minus is applied.

For example, a portable way to express the minimum `long long` value is typically written through the corresponding limit constant or as:

```cpp
-9223372036854775807LL - 1
```

rather than assuming the positive magnitude `9223372036854775808` itself has type `long long`.

<br />

---

<br />

## Querying Limits with `std::numeric_limits`

Hardcoded range assumptions are rarely necessary in generic C++ code.

The `<limits>` header provides `std::numeric_limits<T>` for querying properties of arithmetic types on the current implementation.

```cpp
#include <limits>

constexpr int min{std::numeric_limits<int>::min()};
constexpr int max{std::numeric_limits<int>::max()};
```

Useful members for integral types include:

```cpp
std::numeric_limits<T>::min()
std::numeric_limits<T>::max()
std::numeric_limits<T>::lowest()
std::numeric_limits<T>::digits
std::numeric_limits<T>::digits10
std::numeric_limits<T>::is_signed
std::numeric_limits<T>::is_integer
std::numeric_limits<T>::is_modulo
```

For integer types:

```text
min() == lowest()
```

`digits` reports the number of non-sign value bits for signed integer types and the number of value bits for unsigned integer types.

Example:

```cpp
static_assert(std::numeric_limits<int>::is_integer);
static_assert(std::numeric_limits<unsigned>::is_modulo);
```

> [!info]
> Prefer `std::numeric_limits<T>` when writing templates or generic code. C-style constants such as `INT_MAX` from `<climits>` remain useful when working directly with a known fundamental type.

<br />

---

<br />

## Fixed-Width Integer Types (`<cstdint>`)

When the number of bits is part of the program's contract, C++ provides fixed-width and width-oriented integer aliases in `<cstdint>`.

<br />

### Exact-Width Types

```cpp
#include <cstdint>

std::int8_t
std::uint8_t

std::int16_t
std::uint16_t

std::int32_t
std::uint32_t

std::int64_t
std::uint64_t
```

These types have exactly the stated number of bits and no padding bits **when the implementation provides a suitable underlying integer type**.

> [!important]
> Exact-width aliases such as `std::int32_t` are optional. A conforming implementation is not required to provide `std::int32_t` if it has no integer type satisfying the exact requirements.

On mainstream systems, the familiar 8/16/32/64-bit aliases are normally available.

<br />

### Least-Width Types

The `least` family guarantees at least the requested width while choosing a smallest suitable type:

```cpp
std::int_least8_t
std::int_least16_t
std::int_least32_t
std::int_least64_t

std::uint_least8_t
std::uint_least16_t
std::uint_least32_t
std::uint_least64_t
```

<br />

### Fast-Width Types

The `fast` family provides an implementation-selected type intended to be fast while having at least the requested width:

```cpp
std::int_fast8_t
std::int_fast16_t
std::int_fast32_t
std::int_fast64_t

std::uint_fast8_t
std::uint_fast16_t
std::uint_fast32_t
std::uint_fast64_t
```

A `fast` type is not required to have exactly the number of bits in its name.

For example:

```cpp
std::int_fast16_t
```

may be a 32-bit or 64-bit type if the implementation considers that representation preferable.

<br />

### Maximum-Width Integer Types

```cpp
std::intmax_t
std::uintmax_t
```

These are the maximum-width signed and unsigned integer types provided by the implementation's `<cstdint>` interface.

<br />

### Integer Types Capable of Holding Pointers

Where provided:

```cpp
std::intptr_t
std::uintptr_t
```

are integer types capable of representing converted `void*` pointer values according to their specified guarantees.

These are intended for low-level interoperability scenarios and should not be treated as ordinary replacements for pointers.

<br />

### When Fixed Width Is Appropriate

Fixed-width types are especially useful when width is semantically part of the contract:

- binary file formats;
- network protocols;
- serialization formats;
- hardware registers;
- cryptographic or bit-level algorithms;
- foreign-function or ABI interfaces with explicitly defined widths.

For ordinary counters and arithmetic, `int` may still be exactly the right type.

> [!important]
> Choosing `std::uint32_t` does not by itself define a complete binary serialization format. Byte order, alignment, padding, and encoding rules may still need to be specified explicitly.

<br />

### `std::int8_t` and `std::uint8_t` Caveat

If `std::int8_t` and `std::uint8_t` exist, they are commonly aliases of `signed char` and `unsigned char` because those are the standard integer types capable of satisfying an exact 8-bit requirement on ordinary systems.

This can matter with stream insertion and APIs that treat character types specially.

For example, depending on the alias:

```cpp
std::uint8_t value{65};
std::cout << value;
```

may behave like character output rather than printing the numeric value `65`.

An explicit conversion can make the intent clear:

```cpp
std::cout << static_cast<unsigned int>(value);
```

<br />

---

<br />

## `std::size_t`

`std::size_t`, declared in `<cstddef>`, is an unsigned integer type used for representing object sizes and related quantities.

```cpp
#include <cstddef>

std::size_t n{};
```

The `sizeof` operator returns `std::size_t`:

```cpp
auto bytes{sizeof(int)};
```

The type of `bytes` is `std::size_t`.

`std::size_t` is also commonly used by standard containers for sizes and indices:

```cpp
std::vector<int> values{1, 2, 3};
std::size_t count{values.size()};
```

> [!important]
> `std::size_t` is unsigned. This is part of its type semantics, not a general recommendation that every count or index in user code should be unsigned.

<br />

### C++23 Size Literals

C++23's `uz` / `UZ` literal suffix can directly produce `std::size_t`:

```cpp
std::size_t n{42uz};
```

<br />

---

<br />

## `std::ptrdiff_t`

`std::ptrdiff_t`, declared in `<cstddef>`, is a signed integer type used for pointer differences.

```cpp
#include <cstddef>

int values[10]{};

std::ptrdiff_t distance{&values[8] - &values[2]};
```

The result is:

```text
6
```

Pointer subtraction is meaningful only when the pointers participate in the same array-object relationship required by the language rules.

Because differences can be negative, `std::ptrdiff_t` is signed.

<br />

---

<br />

## `std::ssize` (C++20)

Many standard container `.size()` functions return an unsigned size type.

When a signed size is intentionally more convenient, C++20 provides `std::ssize` in `<iterator>`:

```cpp
#include <iterator>
#include <vector>

std::vector<int> values{1, 2, 3};

auto n{std::ssize(values)};
```

This can reduce accidental signed/unsigned mixing in algorithms that naturally work with signed differences.

<br />

---

<br />

## Platform Data Models

The exact sizes of standard integer types are influenced by the platform ABI and data model, not simply by whether the processor is described as "32-bit" or "64-bit."

Three common data models are:

```text
ILP32
LP64
LLP64
```

<br />

### ILP32

```text
int     = 32 bits
long    = 32 bits
pointer = 32 bits
```

This model is common on many 32-bit systems.

<br />

### LP64

```text
int     = 32 bits
long    = 64 bits
pointer = 64 bits
```

This is common on 64-bit Unix-like systems, including typical modern Linux and macOS environments.

<br />

### LLP64

```text
int       = 32 bits
long      = 32 bits
long long = 64 bits
pointer   = 64 bits
```

Windows x64 uses LLP64.

<br />

### Typical 64-Bit Comparison

| Type | Windows x64 — LLP64 | Linux x86-64 — LP64 | macOS 64-bit — LP64 |
|---|---:|---:|---:|
| `signed char` | 8 | 8 | 8 |
| `unsigned char` | 8 | 8 | 8 |
| `short` | 16 | 16 | 16 |
| `unsigned short` | 16 | 16 | 16 |
| `int` | 32 | 32 | 32 |
| `unsigned int` | 32 | 32 | 32 |
| `long` | **32** | **64** | **64** |
| `unsigned long` | **32** | **64** | **64** |
| `long long` | 64 | 64 | 64 |
| `unsigned long long` | 64 | 64 | 64 |
| pointer | 64 | 64 | 64 |

This means both of the following layouts are normal and valid:

```cpp
// Typical Windows x64
static_assert(sizeof(int)       == 4);
static_assert(sizeof(long)      == 4);
static_assert(sizeof(long long) == 8);
static_assert(sizeof(void*)     == 8);
```

```cpp
// Typical 64-bit Linux/macOS
static_assert(sizeof(int)       == 4);
static_assert(sizeof(long)      == 8);
static_assert(sizeof(long long) == 8);
static_assert(sizeof(void*)     == 8);
```

> [!important]
> Never infer that C++ `long` must be 64 bits merely because the target CPU or process is 64-bit.

<br />

---

<br />

## Common Type-Selection Guidance

There is no single integer type that is best for every purpose.

<br />

### Prefer `int` for Ordinary Integer Arithmetic

When its range is sufficient, `int` is usually the natural default for ordinary arithmetic:

```cpp
int count{10};
int score{250};
int delta{-3};
```

It participates naturally in integral promotions and generally matches the processor's efficient general-purpose arithmetic width on mainstream targets.

<br />

### Use Wider Types When the Range Requires It

If the required mathematical range may exceed `int`, choose a wider type deliberately:

```cpp
long long total{};
```

Do not assume `long` is always wider than `int` in a practically useful way; they may have the same width.

<br />

### Use Unsigned Types for Unsigned Semantics

Unsigned types are appropriate when modulo arithmetic, bit patterns, or an interface contract makes unsigned behavior meaningful.

```cpp
unsigned int flags{};
```

Do not use unsigned solely as a substitute for validating that a value is non-negative.

<br />

### Use Fixed-Width Types When Width Is the Contract

```cpp
std::uint32_t packet_field{};
```

is clearer than assuming:

```cpp
unsigned long packet_field{};
```

has the same width on every platform.

<br />

### Use `std::size_t` for Size-Oriented Interfaces

When storing or passing the result of `sizeof`, container sizes, or APIs explicitly defined in terms of `std::size_t`, use the interface's natural type rather than forcing an unrelated integer type.

<br />

### Avoid Unnecessary Casts Between Signed and Unsigned

A cast can silence a compiler warning without fixing the underlying logic.

Prefer first establishing that the value is in range and that the conversion matches the intended semantics.

```cpp
if (value >= 0) {
    auto converted{static_cast<unsigned int>(value)};
    // use converted
}
```

<br />

---

<br />

## Common Pitfalls

### Assuming `int` Is Always 32 Bits

```cpp
static_assert(sizeof(int) == 4); // platform assumption, not language-wide guarantee
```

This may be reasonable inside a platform-specific codebase, but it is not a portable C++ language assumption.

<br />

### Assuming `long` Is 64 Bits on Every 64-Bit Platform

Windows x64 demonstrates why this assumption fails:

```text
sizeof(long) == 4
sizeof(void*) == 8
```

<br />

### Treating Plain `char` as Reliably Signed

```cpp
char x{-1};
```

code that depends on negative plain-`char` behavior is not portable across implementations with different plain-`char` signedness behavior.

<br />

### Assuming Unsigned Means "Cannot Go Negative"

```cpp
unsigned int x{0};
--x;
```

The value does not remain zero or produce a negative number; it wraps to the maximum value of the unsigned type.

<br />

### Mixing Signed and Unsigned Values Without Considering Conversions

```cpp
int x{-1};
unsigned int y{1};

if (x < y) {
    // may not execute because x can be converted to unsigned
}
```

<br />

### Forgetting Integral Promotions

```cpp
unsigned char a{200};
unsigned char b{100};

auto result{a + b};
```

On ordinary systems, `result` is `int`, not `unsigned char`.

<br />

### Assuming `std::int32_t` Must Exist

Exact-width aliases are conditional on a suitable implementation-provided integer type.

<br />

### Assuming Fixed Width Solves Serialization Completely

```cpp
std::uint32_t value{};
```

defines the width of the integer type when available, but does not automatically specify byte order or an external binary format.

<br />

### Accidentally Writing an Octal Literal

```cpp
int permission{0755}; // octal
```

The leading zero changes the base.

<br />

### Believing a Literal Suffix Must Match the Destination Type

```cpp
long long x{42}; // completely valid
```

The suffix affects the literal's own type-selection process; ordinary initialization conversions can still initialize a different destination type.

<br />

---

<br />

## Useful Headers

### `<climits>`

Provides implementation-specific limits and `CHAR_BIT`:

```cpp
#include <climits>

CHAR_BIT
SCHAR_MIN
SCHAR_MAX
UCHAR_MAX
SHRT_MIN
SHRT_MAX
USHRT_MAX
INT_MIN
INT_MAX
UINT_MAX
LONG_MIN
LONG_MAX
ULONG_MAX
LLONG_MIN
LLONG_MAX
ULLONG_MAX
```

<br />

### `<limits>`

Provides the generic `std::numeric_limits<T>` interface:

```cpp
#include <limits>

std::numeric_limits<int>::min()
std::numeric_limits<int>::max()
```

<br />

### `<cstdint>`

Provides fixed-width and related integer aliases:

```cpp
#include <cstdint>

std::int32_t
std::uint32_t
std::int_least32_t
std::uint_least32_t
std::int_fast32_t
std::uint_fast32_t
std::intmax_t
std::uintmax_t
std::intptr_t
std::uintptr_t
```

where the optional aliases are available when the implementation satisfies their requirements.

<br />

### `<cstddef>`

Provides size- and pointer-difference-related types:

```cpp
#include <cstddef>

std::size_t
std::ptrdiff_t
```

<br />

### `<utility>`

Provides C++20 safe mixed-signedness integer comparison helpers:

```cpp
#include <utility>

std::cmp_equal
std::cmp_less
std::cmp_greater
```

<br />

---

<br />

## Mastery Checklist

A strong understanding of integral types means being able to explain all of the following without relying on platform assumptions:

- why `char`, `signed char`, and `unsigned char` are distinct types;
- why plain `char` cannot be assumed to be signed;
- why `sizeof(char)` is always `1` but a byte is not guaranteed to contain exactly 8 bits;
- the minimum size/range relationships among `short`, `int`, `long`, and `long long`;
- why a 64-bit process does not imply a 64-bit `long`;
- the difference between LP64 and LLP64;
- why signed overflow is undefined while unsigned arithmetic is modular;
- why using unsigned does not enforce a non-negative program invariant;
- how integral promotions change operations on `char` and `short`;
- why mixed signed/unsigned arithmetic can produce surprising values;
- the purpose of integer conversion rank;
- how conversions to unsigned types behave modulo `2^N`;
- why out-of-range signed conversion differs from signed arithmetic overflow;
- how brace initialization helps reject narrowing conversions;
- how decimal, octal, hexadecimal, and binary integer literals are written;
- how integer-literal base and suffix influence the literal's type;
- why `42`, `42U`, `42L`, and `42LL` are different typed expressions;
- why the minus sign is not part of an integer literal;
- when `std::numeric_limits<T>` should be preferred over hardcoded limits;
- what the exact-, least-, and fast-width `<cstdint>` families mean;
- why `std::int32_t` is not guaranteed to exist on every conforming implementation;
- why `std::int8_t` / `std::uint8_t` may behave like character types in I/O;
- what `std::size_t` represents and why it is unsigned;
- what `std::ptrdiff_t` represents and why it is signed;
- when `std::ssize` can reduce signed/unsigned friction;
- and how to choose an integer type according to **semantics, required range, portability, and interface contracts** rather than habit.

<br />

---