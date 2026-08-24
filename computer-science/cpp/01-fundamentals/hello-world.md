# Hello, World

Below is an example of the classic [hello, world](https://en.wikipedia.org/wiki/Hello,_world) implemented in C++.

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!\n";
}
```

<br />

---

<br />

## Part 1: Breakdown

```cpp
#include <iostream>
```

- This is a [preprocessor](https://en.cppreference.com/cpp/preprocessor) directive. The preprocessor runs before compilation.
- `#include <iostream>` literally copies the contents of the header file `iostream` into your source file.
- Including `<iostream>` gives you access to objects such as `cin`, `cout`, `cerr`, `clog`, as well as their wide character variants (`wcin`, `wcout`, `wcerr`, `wclog`). 
- Reference: [iostream](https://cplusplus.com/reference/iostream/)

> [!note]
> Angle brackets `< >` are used for standard library header files or external, third-party libraries.
> 
> Use double quotes `" "` for your local header files that belong to your specific project.

<br />

---

<br />

## Part 2: Breakdown

```cpp
int main()
```

- `main` is the entry point of every C++ program.
- The runtime calls `main` after initializing global/static objects.
- The return type is `int`. The value returned is sent back to the operating system as the program’s exit status.
- `()` means the function takes **no parameters**. This is the first of the two standard forms of `main`.
- The body is enclosed in `{ }`. This is a **function definition**.

<br />

---

<br />

## Part 3: Breakdown

```cpp
std::cout
```

- `std::cout` is an object of type `std::ostream` that represents the standard output stream (usually the terminal).
- It lives in the `std` namespace, hence the `std::` prefix.


> [!note]
> Avoid `using namespace std;` at global scope. It pulls the entire standard library into the global namespace, which can cause name collisions.
> 
> Either use the `std::` prefix or put `using std::cout;` inside a function.

<br />

---

<br />

## Part 4: Breakdown

```cpp
<<
```

- Stream insertion operator:
- `<<` is an operator overload for output streams.
- It returns a reference to the stream (`std::ostream&`), which allows chaining:

```cpp
std::cout << "Hello, " << "World!" << '\n';
```

<br />

---

<br />

## Part 5: Breakdown

```cpp
"Hello, World!\n"
```

- This is a string literal. Its type is `const char[14]` because the string contains 13 visible characters plus a hidden null terminator `\0`.
- `\n` is an escape sequence for a newline. It is a single character in the string (this is covered in more detail during the characters section of the roadmap). 

<br />

---

<br />

## Part 6: Breakdown

```cpp
;
```

- Every statement in C++ ends with a semicolon.


> [!warning]
> In C++, semicolons are mandatory. Forgetting one will cause a compile-time error, and your code will not compile.

<br />

---

<br />

## Modern Best Practices

<br />

1. Use `'\n'`, not `std::endl`

```cpp
// good
std::cout << "Hello, World!\n";

// also good
std::cout << "Hello, World!" << '\n';

// unnecessary flush
std::cout << "Hello, World!" << std::endl;
```

<br />

2. `return 0;` is optional in a `main` function:

If control reaches the end of the [`main` function](https://en.cppreference.com/cpp/language/main_function "cpp/language/main function"), `return 0;` is executed (even if not explicitly written).

So these examples execute similarly: 

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!\n";
}
```

```cpp
# include <iostream>

int main() {
    std::cout << "Hello, World!\n";
    return 0; // <--
}
```

<br />

3. Readability & Redundancy

In some codebases, you may see `EXIT_SUCCESS` being used instead of `0`:

```cpp
#include <iostream>
#include <cstdlib>

int main() {
	std::cout << "Hello, World!\n";
	return EXIT_SUCCESS;
}
```

While technically correct, using `EXIT_SUCCESS` in `main()` is not recommended as best practice for modern C++ due to:

- **Implicit Return**: The compiler automatically inserts `return 0;` if omitted.
- **Universal Standard**: Returning `0` is universally understood as success, and the C++ standard guarantees the compiler will translate `0` to the host platform's native success code.
- **Redundant Header**: It requires including `<cstdlib>`, adding an unnecessary dependency to the file.

> [!info]
> Developers typically only use `EXIT_SUCCESS` for visual consistency when a program has multiple exit paths that require `EXIT_FAILURE`.

<br />

---

<br />

## \<print> (C++23)

If your compiler supports C++23, you can use `<print>`:

```cpp
#include <print>

int main() {
	std::println("Hello, World!");
}
```

`std::println` is type‑safe, often faster, and automatically appends a newline. But it is not yet universal, so `std::cout` remains the portable fallback.

<br />

There is also `std::print` which does the exact same thing as `std::println`, however it does not append a newline:

```cpp
#include <print>

int main() {
	std::print("Hello, ");
	std::print("World!");
	
	// output: Hello, World!
}
```

<br />

You must compile using the appropriate compiler flags to use `<print>`. 

| Toolchain Ecosystem | Minimum Compiler | Minimum Library | Required Flag | 
| :-- | :-- | :--: | :-- |
| GNU (Linux/MinGW) | GCC 14.1.0 | libstdc++ 14.1 | `-std=c++23` |
| LLVM (Generic) | Clang 18.0 | libc++ 18.0 | `-std=c++23` |
| Microsoft (Windows) | MSVC 19.37 | VS 2022 v17.7 | `/std:c++23` |
| Apple (Mac) | Apple Clang 16.0 | Xcode 16 SDK | `-std=c++23` |


> [!note]
> If your compiler does not yet support <print>, or you are restricted to a specific environment, <iostream> should be used instead.

<br />

---

<br />

## Main Signatures

The C++ standard defines exactly two signatures for main that must be supported by every implementation:

First signature:

```cpp
int main() {}
```

Second signature:

```cpp
int main(int argc, char* argv[]) {};
```

<br />

### Difference

```cpp
main()
```

- declares that the program takes **no command‑line arguments** (or, more precisely, that you do not need to access them).
- The operating system still passes arguments, but they are ignored.

```cpp
main(int argc, char* argv[])
```

- declares parameters to receive command‑line arguments.

<br />

### What do argc and argv mean?

`argc` = **argument count**: the number of command‑line arguments, including the program name.

`argv` = **argument vector**: an array of C‑style strings (char*) holding the arguments.

- `argv[0]` is the program name (or an empty string if not available).
- `argv[1]` ... `argv[argc-1]` are the actual arguments.
- `argv[argc]` is guaranteed to be a null pointer (nullptr).

<br />

### Example

if you run: 

```sh
./program one two three
```

then: 

- `argc == 4`
- `argv[0] == "./program"`
- `argv[1] == "one"`
- `argv[2] == "two"`
- `argv[3] == "three"`
- `argv[4] == nullptr`