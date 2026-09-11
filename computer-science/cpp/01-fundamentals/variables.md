# C++ Variables

A **variable** gives a program a name through which it can access an object or reference.

To reason about a variable correctly, keep these ideas separate:

- **Type**: what kind of value/object it represents and which operations are valid.
- **Initialization**: how its initial value or state is established.
- **Mutability**: whether the program may modify it.
- **Scope**: where its name is visible.
- **Storage duration**: how long its storage persists.
- **Lifetime**: when the object actually exists.
- **Linkage**: whether declarations in different places can refer to the same entity.

<br />

> [!important]
> Scope, storage duration, lifetime, and linkage are related concepts, but they are **not interchangeable**.

<br />

---

<br />

## Declaration vs. Definition vs. Initialization

<br />

### Declaration

A **declaration** introduces or redeclares an entity and gives the compiler information about it, such as its name and type.

```cpp
extern int player_level;
```

This declares `player_level`, but does not define it.

<br />

### Definition

A **definition** is a declaration that fully defines the entity.

```cpp
int player_level;
```

This is both a **declaration** and a **definition**.

<br />

> [!info]
> **Key Rule:** Every definition is a declaration, but not every declaration is a definition.

<br />

### Initialization

**Initialization** establishes an object's initial value or state when the object is created.

```cpp
int player_level{1};
```

This statement:

1. declares `player_level`,
2. defines `player_level`,
3. initializes it to `1`.

Initialization is different from assignment:

```cpp
int score{10}; // initialization
score = 20;    // assignment
```

<br />

---

<br />

## Prefer Initialized Variables

A strong modern C++ default is to give a variable a meaningful initial value when it is defined.

```cpp
int count{0};
double temperature{0.0};
bool active{false};
```

<br />

Avoid leaving ordinary local scalar variables uninitialized without a deliberate reason:

```cpp
int count; // avoid when no valid initial state is intended
```

Reading an indeterminate value can lead to incorrect behavior and, depending on the situation, undefined behavior.

<br />

> [!tip]
> **Best Practice:** Define a variable when you have a sensible initializer for it instead of declaring it long before it can receive a value.

<br />

---

<br />

## Initialization Syntax: Essential Overview

<br />

### Direct List Initialization

```cpp
int x{32};
std::string name{"Ada"};
```

<br />

Braces are an excellent default because they reject many narrowing conversions:

```cpp
int x{7.9}; // error: narrowing conversion
```

### Value Initialization

Empty braces are useful when a value-initialized object is desired:

```cpp
int count{};    // 0
double ratio{}; // 0.0
bool active{};  // false
```

### Copy Initialization

```cpp
int x = 32;
std::string name = "Ada";
```

Despite the `=` token, this is **initialization**, not assignment.

### Parenthesized Direct Initialization

```cpp
std::string name("Ada");
```

<br />

Parentheses remain important when constructor semantics differ from braces:

```cpp
std::vector<int> a(10, 20); // 10 elements, each equal to 20
std::vector<int> b{10, 20}; // 2 elements: 10 and 20
```

<br />

> [!important]
> **Prefer `{}` as a default, not as an absolute rule.**
> List initialization gives special priority to viable `std::initializer_list` constructors, so `()` may be necessary when you intend another constructor overload.

<br />

---

<br />

## Const (Immutability)

Use `const` when a variable should not be modified after initialization.

```cpp
const int max_players{100};
```

```cpp
max_players = 200; // error
```

<br />

`const` does **not** mean that the value was necessarily computed at compile time:

```cpp
const int user_choice{read_choice()}; // may be initialized at runtime
```

<br />

> [!tip]
> **Best Practice:** Prefer `const` when mutation is not required.

<br />

---

<br />

## Constexpr (Compile-Time Constant Variables)

A `constexpr` variable must satisfy the requirements for constant evaluation and is also `const`.

```cpp
constexpr int max_players{64};
constexpr double pi{3.141592653589793};
```

<br />

It can be used where C++ requires a constant expression:

```cpp
constexpr int array_size{8};
int values[array_size]{};
```

<br />

> [!info]
> Think of `constexpr` as a **language guarantee about constant-expression usability**, not merely as an optimization request.

<br />

---

<br />

## Constinit (Static Initialization (C++20))

`constinit` applies to variables with **static or thread storage duration** and requires static initialization.

```cpp
constinit int global_counter{0};
```

<br />

It does **not** make the variable immutable:

```cpp
global_counter = 10; // allowed
```

<br />

---

<br />

## Auto (Type Deduction)

`auto` asks the compiler to deduce a variable's type from its initializer.

```cpp
auto count{32};         // int
auto ratio{3.14};       // double
auto text{"Ad astra"};  // const char*
```

<br />

Because deduction requires an initializer:

```cpp
auto value; // error
```

<br />

References and qualifiers affect deduction:

```cpp
int value{42};
const auto& ref{value}; // const int&
```

<br />

Brace deduction has a few special rules:

```cpp
auto a{5};       // int
auto b = {1, 2}; // std::initializer_list<int>
auto c{1, 2};    // error
```

<br />

> [!tip]
> Use `auto` when the type is obvious from the initializer or spelling the exact type would add noise. Prefer an explicit type when the type itself communicates important meaning.

<br />

---

<br />

## Scope, Storage Duration, and Lifetime

<br />

### Scope

**Scope** determines where a **name** can be used.

```cpp
void example() {
    int x{10};

    if (x > 0) {
        int y{20}; // y is visible only inside this block
    }
}
```


### Storage Duration

**Storage duration** describes how long an object's storage persists.

C++ defines four storage-duration categories:

- **automatic**
- **static**
- **thread**
- **dynamic**

Do not treat "stack" and "heap" as exact synonyms for automatic and dynamic storage duration. They are useful implementation terms, but the C++ language model is defined in terms of storage duration.

### Lifetime

**Lifetime** describes the period during which an object actually exists and may be used as that object.

Storage duration and lifetime are therefore separate concepts.

### Scope Does Not Determine Storage Duration

```cpp
void count_calls() {
    int current{};     // block scope, automatic storage duration
    static int total{}; // block scope, static storage duration

    ++current;
    ++total;
}
```

Both variables have block scope, but their storage durations differ.

---

## Linkage and `extern`

**Linkage** determines whether declarations in different scopes or translation units can refer to the same entity.

A traditional cross-file variable uses `extern` for a declaration and one definition elsewhere:

```cpp
// game_state.hpp
extern int player_level;
```

```cpp
// game_state.cpp
int player_level{1};
```

For header-defined constants, modern C++ can use an `inline constexpr` variable:

```cpp
// limits.hpp
inline constexpr int max_players{64};
```

<br />

> [!warning]
> Mutable global state creates hidden dependencies and is usually harder to reason about, test, and synchronize. Prefer narrower ownership and explicit dependencies when practical.

<br />

---

<br />

## Variables and Value Categories Are Different Concepts

A variable and an expression are not the same thing.

```cpp
int x{5};
```

`x` is a variable. When the name `x` appears in an expression, that **expression** has a value category.

```cpp
x;     // lvalue expression
x + 1; // typically a prvalue expression
```

<br />

> [!important]
> **Variables do not themselves have value categories; expressions do.**

<br />

---

<br />

## Modern Variable Best Practices

1. **Initialize variables when they are defined.**
2. **Declare variables close to first use.**
3. **Keep scope as narrow as practical.**
4. **Prefer `const` unless mutation is required.**
5. **Use `constexpr` when a value is genuinely compile-time constant.**
6. **Use `auto` when deduction improves clarity rather than hiding important type information.**
7. **Avoid accidental variable shadowing.**
8. **Do not reuse one variable for unrelated meanings.**
9. **Avoid mutable global state when practical.**
10. **Prefer names that describe a variable's role rather than merely its type.**

Example of narrow scope and immediate initialization:

```cpp
if (const auto result{find_value()}; result.has_value()) {
    use(*result);
}
```

`result` is initialized immediately, cannot be modified, and exists only where it is needed.

<br />

---

<br />

## Mental Model

When reasoning about a variable, ask each question separately:

1. **What is its type?**
2. **Where is it declared and defined?**
3. **How is it initialized?**
4. **Can it be modified?**
5. **Where is its name visible?** → scope
6. **How long does its storage persist?** → storage duration
7. **When does the object actually exist?** → lifetime
8. **Can declarations elsewhere refer to the same entity?** → linkage
9. **Is its type explicit or deduced?** → type deduction

Keeping these concepts separate prevents many of the mistakes that make later C++ topics difficult.