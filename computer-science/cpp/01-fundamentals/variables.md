# C++ Variables

a **variable** is a named memory location that holds a value. Mastering variables requires understanding **how they are stored, when they are born and destroyed, where they can be accessed, and how they should be initialized.**

<br />

---

<br />

## Declaration vs. Definition vs Initialization

- **Declaration**: Announces a variable's **name** and its data **type** to the compiler so it can be used in code. It allocates **no memory**.

```cpp
extern int age;
```

- **Definition**: Creates the variable and **allocates memory** for it. It fulfills the promise made by a declaration.

```cpp
int age;
```

- **Initialization**: Fills that memory with an initial value.

```cpp
int age = 32;
```

> [!info]
> **Key Rule:** Every definition is implicitly a declaration, but not every declaration is a definition.

<br />

---

<br />

## Local Variables vs. Global/Static Variables

For **ordinary local variables** inside functions, declaring and defining happen at the exact same time:

```cpp
void example() {
	int x; // Declaration AND Definition (allocates memory)
	int y{32}; // Declaration AND Definition AND Initialization (allocates memory)
}
```

> [!info]
> However, the distinction becomes explicit when working across multiple files, with `static` class members, or with global variables.


<br />

---

<br />

## Pure Declarations using extern

When you want to share a variable across multiple `.cpp` files, you need to declare it in a header file without allocating memory, and define it in exactly one source file.

```cpp
// Header.hpp
extern int player_level; // PURE DECLARATION
```

```cpp
// main.cpp
int player_level{1}; // DEFINITION (allocates memory for declared variable)
```

<br />

---

<br />

## Lifetime, Scope, and Storage

Every variable in C++ is defined by three fundamental properties:

- **Scope:** The region of code where the variable's name is visible (e.g., Block `{...}`, Function, File/Global, Namespace).
- **Storage Duration (Lifetime):** Controls when the memory is allocated and destroyed.
    - `automatic`: Local variables (created on entry to a block, destroyed on exit via the stack).
    - `static`: Allocated at program start, destroyed at exit.
    - `dynamic`: Allocated manually on the heap via `new` / smart pointers.
    - `thread_local`: Unique per thread.
        
- **Value Category:** Whether the variable represents an identity (lvalue) or a temporary value (rvalue).

<br />

---

<br />

## Ways to Initialize Variables in Modern C++

C++ has evolved over decades, resulting in multiple syntax styles for initialization. Understanding all of them, and knowing which one to pick, is critical.

<br />

### Direct Initialization

```cpp
int x(32);
```

```cpp
std::string s("Ad astra");
```

- **How it works:** Uses parentheses to pass arguments directly to a constructor or scalar initializer.
- **Drawback:** Vulnerable to the **"Most Vexing Parse"** (where the compiler confuses a variable declaration for a function declaration).

<br />

### Copy Initialization

```cpp
int x = 32;
```

```cpp
std::string s = "Ad astra";
```

- **How it works:** Looks like assignment, but creates a temporary and copies/moves it into the variable (though modern compilers optimize the copy away via copy elision).
- **Drawback:** Implicit conversions can happen silently (e.g., `int x = 3.14;` compiles cleanly and truncates to `3`).

<br />

### Value Initialization (Defaulting)

```cpp
int x{}; // x defaults to 0
```

```cpp
double y{}; // y defaults to 0.0
```

```cpp
bool z{}; // z defaults to false
```

**How it works:** Empty braces zero-initialize primitives and call the default constructor for objects.

<br />

### Direct List Initialization

Introduced in C++11 to unify all initialization syntax (also called **Brace Initialization** or **Uniform Initialization**).

```cpp
int x{32};
```

```cpp
std::string s{"Ad astra"};
```

```cpp
std::vector<int> v{1, 2, 3};
```

<br />

---

<br />

## Why Brace Initialization is Best Practice

Modern C++ guidelines strongly recommend **Brace Initialization `{}`** (also known as Direct List Initialization) as your default choice. Here is why:

### Prevents Narrowing Conversions (Compile-Time Safety)

Traditional initialization allows silent data loss through implicit conversion:

```cpp
// Traditional (Compiles with silent data loss)
int a = 7.9; // a becomes 7
```

```cpp
// Uniform Initialization (Compilation Error)
int b{7.9}; // ERROR: narrowing conversion from 'double' to 'int'
```

<br />

### Solves the "Most Vexing Parse"

```cpp
std::string s(); // declares a function!
std::string s{}; // creates an object
```

In traditional C++, this line is ambiguous:

```cpp
// Is this creating a Time object 't' with a default timer,
// or declaring a function named 't'?
Time t(Timer());
```

With braces, there is zero ambiguity:

```cpp
Time t{Timer{}}; // Guaranteed to be a variable declaration
```

<br />

### Provides a Single, Consistent Syntax

Braces work for scalars, objects, arrays, vectors, and aggregate types:

```cpp
int x{32};
int arr[]{1, 2, 3};
std::vector<int> nums{1, 2, 3};
Point p{10, 20}; // Struct aggregate
```

<br />

### Edge Cases: When Brace Initialization Can Surprise You

While brace initialization is the modern standard, you must be aware of two specific caveats:

### Caveat 1: `std::initializer_list` Hijacking

If a class has a constructor that accepts a `std::initializer_list`, **brace initialization will always prefer it**, even if another constructor matches better.

```cpp
// std::vector has a constructor for (size, initial_value) AND (std::initializer_list)

std::vector<int> v1(10, 20); // Creates a vector of 10 elements, all set to 20
std::vector<int> v2{10, 20}; // Creates a vector of 2 elements: [10, 20]
```

> [!important]
> **Rule:** Use parentheses `()` when calling constructors that specify container sizes or capacities rather than element lists.

<br />

### Caveat 2: Auto Type Deduction with Braces

In C++11, `auto x{5};` deduced `std::initializer_list<int>`. C++17 fixed this, but the rules are good to know:

```cpp
auto a = {1, 2}; // std::initializer_list<int>
```

```cpp
auto b{5}; // int (C++17 onwards)
```

```cpp
auto c{1, 2}; // ERROR in C++17 (multi-element direct list init with auto is invalid)
```

<br />

---

<br />

## Variable Mutability and Qualifiers

<br />

`const` (Runtime Read-Only): 
- Promises that the variable's value will not change after initialization.

```cpp
const int max{100};
```

```cpp
max = 200; // Compile error
```

<br />

`constexpr` (Compile-Time Constant):
- Enforces that the value **must** be computable at compile-time. This allows the compiler to optimize code by replacing the variable directly with its calculated value.

```cpp
constexpr double pi{3.1415926535};
constexpr int square(int x) { return x * x; }
constexpr int result = square(5); // Evaluated at compile time!
```

`consteval` (C++20 Immediate Functions/Variables):
- Guarantees execution happens strictly at compile-time; runtime execution is impossible.

<br />

`auto` (Type Inference):
- Tells the compiler to deduce the variable's type from its initializer. **`auto` forces initialization**—you cannot declare an uninitialized `auto` variable.

```cpp
auto x{32}; // int
```

```cpp
auto y{3.14}; // double
```

```cpp
auto z{"Ad astra"}; // const char*
```

```cpp
const auto& ref{x}; // const int&
```